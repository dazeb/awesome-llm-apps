# Give your agents tools that scale down to zero or up to infinity

An agent is only as capable as its tools, and in production those tools run as Model Context Protocol (MCP) servers. Every other layer of your stack scales to zero when it goes idle. Your tools could not, because until last month a single MCP session pinned every tool call to one container.

By [@zeroasterisk](https://x.com/@zeroasterisk) and [@Saboo_Shubham_](https://x.com/@Saboo_Shubham_)

When we began deploying MCP servers across our cloud infrastructure at Google, we hit that wall immediately. Connecting over HTTP meant running an initialize handshake, negotiating capabilities, capturing a session token, and routing every subsequent request back to the exact same pod. The original transport was designed for one client talking to a local sidecar process. Pushing that into a cloud-native environment meant sticky sessions, a Redis cluster whose only job was remembering routing headers, and idle containers we paid for around the clock because they could never safely scale to zero.

The 2026-07-28 Model Context Protocol specification fixes this at the source. It deletes transport sessions, makes every tool call independent, and lets your agent's tools scale to zero on serverless platforms like Cloud Run. Google helped drive the transport changes through the MCP specification process, alongside Hugging Face and other partners.

Here is how the architecture works, and how to wire it into your own agents:

* Sessions were an unadvertised tax on your agent's tools. Why round-robin routing broke older MCP servers with 400 Session Not Found.
* The 2026 spec makes every tool call self-describing. How SEP-2575 and SEP-2567 deleted the handshake and moved identity into inline metadata.
* Tool containers can finally scale down to zero. Deploying stateless MCP tools to Cloud Run and retiring the Redis session cluster.
* Wiring stateless tools into ADK takes six lines of Python. Pointing Google's Agent Development Kit at a streamable HTTP endpoint with no session choreography.
* Stateless is the transport, not your application. How explicit task handles replace hidden connection state for long-running work.

## 1. Sessions were an unadvertised tax on your agent's tools

If you have deployed an MCP server to Kubernetes or Cloud Run, you have probably hit this: your agent initializes cleanly, runs its first tool call, then crashes on the second one with an unhelpful session error.

Under the old transport (version 2025-11-25), an HTTP connection started with a stateful handshake. The client sent an initialize payload, negotiated capabilities, and got back an Mcp-Session-Id header it had to attach to every later request. But cloud load balancers spread requests round-robin, and none of them know which container holds which in-memory session. That is the conflict.

You can reproduce the failure with two curl commands against any older, stateful MCP server on Cloud Run:

```shell
gcloud run deploy mcp-stateful-demo \
  --image=ghcr.io/github/github-mcp-server:v0.6.0 \
  --region=us-central1 \
  --min-instances=0 \
  --allow-unauthenticated
```

Step 1: The agent initializes a connection and extracts the session ID:

```shell
SESSION_ID=$(curl -s -X POST https://mcp-stateful-demo-xyz.a.run.app/mcp \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2025-11-25",
      "capabilities": {},
      "clientInfo": {"name": "adk-agent", "version": "1.0"}
    }
  }' -D - | grep -i "mcp-session-id" | awk '{print $2}' | tr -d '\r')
```

Step 2: Two seconds later, the agent calls a tool using that session ID:

```shell
curl -X POST https://mcp-stateful-demo-xyz.a.run.app/mcp \
  -H "Mcp-Session-Id: $SESSION_ID" \
  -H "Content-Type: application/json" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/call",
    "params": {
      "name": "search",
      "arguments": {"q": "pull_requests"}
    }
  }'
```

If a load balancer routed that second call to a new instance, or if the container restarted, or if the idle container scaled to zero while the agent was processing its prompt, you got this:

```json
{
  "jsonrpc": "2.0",
  "id": 2,
  "error": {
    "code": -32001,
    "message": "Session not found: 1868a90c-3a3f-4f5b"
  }
}
```

There were two workarounds, both bad. Turn on session affinity at your ingress, which creates hot spots and fights your autoscaler. Or run a shared Redis cache to hold session headers across replicas. Either way you keep idle pods warm just so an HTTP header does not get lost.

How to verify it: Check your existing MCP servers for session handling. If your code checks for an incoming session header before executing a tool, you are paying this routing tax today.

## 2. The 2026 spec makes every tool call self-describing

The 2026-07-28 Model Context Protocol specification deletes the session requirement at the transport layer entirely.

Two Specification Enhancement Proposals (SEPs) drove this change:

* SEP-2575 removes the initialize and initialized handshake.
* SEP-2567 removes the Mcp-Session-Id header and protocol-level sessions.

Now every request describes itself. Protocol version, client capabilities, and client identity ride in an inline _meta object on each call. If a client needs to see what a server offers first, a new server/discover method handles that, with no session to set up.

Routing headers are standardized too: Mcp-Protocol-Version, Mcp-Method, and Mcp-Name. Gateways and edge proxies can route, audit, and rate-limit tool calls straight from the headers, without parsing the JSON body.

Here is a stateless tool call on the wire:

```text
POST /mcp HTTP/1.1
Host: mcp.internal.net
Mcp-Protocol-Version: 2026-07-28
Mcp-Method: tools/call
Mcp-Name: search
Content-Type: application/json

{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "tools/call",
  "params": {
    "name": "search",
    "arguments": { "q": "pull_requests" },
    "_meta": {
      "io.modelcontextprotocol/protocolVersion": "2026-07-28",
      "io.modelcontextprotocol/clientCapabilities": {},
      "io.modelcontextprotocol/clientInfo": {
        "name": "adk-agent",
        "version": "1.0"
      }
    }
  }
}
```

Because every request carries its own context, any healthy instance behind your load balancer can serve it. Google shipped version 1.7.0 of the [Go MCP SDK](https://github.com/modelcontextprotocol/go-sdk) on launch day with full stateless support, and it runs in production behind services like the GitHub MCP Server.

How to verify it: Update your client headers. Injecting _meta and standard routing headers allows your requests to pass through ordinary API gateways without special session plugins.

## 3. Tool containers can finally scale down to zero

Once a server holds no in-memory session, hosting gets simple. No dedicated VMs, no warm pool of pods. You run the tools as a serverless workload on Cloud Run.

Deploy your tool container with --min-instances=0:

```shell
gcloud run deploy mcp-github-server \
  --image=ghcr.io/github/github-mcp-server:latest \
  --platform=managed \
  --region=us-central1 \
  --min-instances=0 \
  --max-instances=10 \
  --allow-unauthenticated
```

While your agent is reasoning, waiting on the user, or running local steps, the tool server sits at zero instances and costs you nothing. The moment a tool call arrives, Cloud Run cold-starts a container on demand, serves the call, and scales back down. If fifty agents fire tools at once, it fans out behind plain round-robin routing.

Autoscaling and restarts are now invisible to the agent. Recycle a container mid-release and the load balancer just sends the next call to any ready replica, because there is no session to lose. You can also retire the Redis session cluster. The GitHub MCP Server did on the 2026-07-28 upgrade, cutting a database write on setup and a read on every tool call.

How to verify it: Take an existing MCP tool container, set its session generator to undefined, and deploy it to Cloud Run with --min-instances=0.

## 4. Wiring stateless tools into ADK takes six lines of Python

Connecting an agent to a remote stateless MCP server in Google's Agent Development Kit (ADK) needs no session management. You configure an McpToolset with StreamableHTTPConnectionParams and hand it to the agent.

Because the agent keeps no socket and no handshake open with the server, tool connections survive autoscaling, restarts, and cold starts on their own.

```python
from google.adk.agents import Agent
from google.adk.tools.mcp_tool import McpToolset
from google.adk.tools.mcp_tool.mcp_session_manager import StreamableHTTPConnectionParams

# Connect to the stateless MCP server running on Cloud Run
github_tools = McpToolset(
    connection_params=StreamableHTTPConnectionParams(
        url="https://mcp-github-server-xyz.a.run.app/mcp"
    )
)

# Root agent uses the MCP toolset like any standard local tool
root_agent = Agent(
    name="triage_agent",
    model="gemini-2.5-flash",
    instruction="Triage incoming GitHub issues and assign reviewers.",
    tools=[github_tools],
)
```

There are no keepalive loops and session renewal tokens. The agent simply dispatches an HTTP request when it needs a tool and parses the response when it arrives.

How to verify it: Replace your local stdio tool configs in staging with a remote StreamableHTTPConnectionParams URL. Your agent code stays identical whether the tool is running locally or across the network.

## 5. Stateless is the transport, not your application

A stateless protocol does not mean your application cannot hold state. It means the transport stopped hiding that state in connection memory. The state did not vanish, it moved into explicit application handles.

Some tool calls take forty-five seconds. If your agent asks a server to build a heavy report or export a database snapshot, holding the HTTP socket open for the whole run blocks the agent loop and invites a timeout.

The 2026-07-28 spec formalizes the Tasks extension for this. When the agent calls a long-running tool, the server returns a task handle right away:

```json
{
  "taskId": "task_89f2a4bc",
  "status": "working",
  "pollIntervalMs": 2000
}
```

The server runs the job in the background. The agent takes the task ID, keeps talking to the user, and checks progress with ordinary tasks/get calls. Your workers still store task state and results in a real datastore, and the Google Developers Blog write-up still uses Redis for exactly that. The transport went stateless. Your data layer did not have to.

Keep the boundary clear:

1. Tool servers scale to zero: The MCP servers executing tool requests are stateless over HTTP and scale down when idle.
2. Agents and backends do not: The agent harness tracking conversation history and the databases tracking background tasks remain stateful.

Decouple the transport from session state and your tool layer scales like any web service, while your app keeps full control of the data that matters.

How to steal it: if a tool takes longer than five seconds, do not raise your HTTP timeout. Return a task ID immediately and let the agent poll for the result.

## Making the tool layer boring

The agent is still the star. The stateless spec just made its tool layer cheap, reliable, and elastic. When tools are tied to sessions, every deploy, scale event, and network hiccup can break the agent loop. Delete the handshake, move the metadata into the payload, and load balancing, autoscaling, and failover for your tools become ordinary cloud problems you solve the ordinary way.

Build the simple thing first. Let the tools scale down when no one needs them, and let Cloud Run absorb the burst when your agents get to work.

## Get started today

You can test this architecture in your own staging environment today:

1. Deploy a stateless MCP server to Cloud Run with --min-instances=0.
2. Connect an ADK agent using StreamableHTTPConnectionParams.
3. Hammer the endpoint with concurrent queries to watch it fan out, then leave it idle to watch it scale to zero.

The SDK betas supporting the 2026-07-28 specification are live across all major languages:

* Go SDK: [v1.7.0](https://github.com/modelcontextprotocol/go-sdk/releases/tag/v1.7.0) (ready on launch day and powering the GitHub MCP Server).
* Python SDK: pip install "mcp[cli]==2.0.0b1".
* TypeScript SDK: Modular v2 packages (@modelcontextprotocol/server@beta, @modelcontextprotocol/client@beta).
* Specification: [2026-07-28 Release Candidate](http://modelcontextprotocol.io/specification/2026-07-28).
* ADK Documentation: [Google Agent Development Kit MCP Toolset](https://adk.dev/tools-custom/mcp-tools/).

Your agent's tools cost nothing when no one is using them, and scale out the second it does.

If you found this breakdown useful, check out our previous article on [5 design patterns for long-horizon agent harness](https://x.com/GoogleCloudTech/status/2090248297214525569). Follow [@GoogleCloudTech](https://x.com/@GoogleCloudTech) for more architectural deep dives from the builder trenches.
