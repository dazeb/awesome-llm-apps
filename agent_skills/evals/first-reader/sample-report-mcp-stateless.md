# First-reader report: "Give your agents tools that scale down to zero or up to infinity"

Draft: the MCP stateless-transport article (1,570 prose words plus code), co-authored for the Google Cloud Tech audience. Venue: Google Cloud blog and long-form X. Intended reader: engineers running agents and MCP servers in production. Intended gist, inferred and used as the recall benchmark: the reader understands the 2026-07-28 spec removed transport sessions so MCP tool servers can scale to zero, and tries the Cloud Run plus ADK setup in staging.

Contamination disclosure: the author pasted the draft into my context before the review, so I was contaminated as a reader from the start. Every reading step below was performed by fresh subagents that saw only what the scripts released; my own full-text judgment came last, as the skill requires. One reader agent died mid-run on a transient API error and was re-dispatched once on a clean session.

Cast: "the platform engineer," sympathetic, owns an MCP deployment currently held together with ingress session affinity and a resented Redis cache, sent the link by a teammate, must defend whatever they bring to Thursday's infra review. "The infra skeptic," staff engineer via X, has run MCP in production since early 2025, allergic to vendor content marketing, with three standing questions: auth and identity, server-initiated messages, and migration for existing stateful servers.

## 1. Sayback and meaning

As received, this piece is a spec explainer wearing a light platform pitch: the 2026 MCP revision deleted transport sessions, and the piece proves it with checkable specifics, teaches two genuinely new things, and closes honestly about where state still lives. Both readers finished all 18 chunks willingly. That is the headline: this piece works, and the findings below are about where it leaks trust, not whether it holds attention.

What landed, in the readers' own logs:

- The pain recognition in paragraph two. The platform engineer: "sticky sessions at ingress, Redis holding routing state, containers that never idle down. That's my exact bill." The skeptic: "That's literally my architecture diagram." Both committed on recognition, not on promises.
- The checkable specifics as a trust engine. SEP-2575 and SEP-2567 by number, the -32001 error payload ("burned into my retinas from our own logs"), the pinned Go SDK v1.7.0, the GitHub MCP Server Redis retirement with its cost delta. The skeptic said it outright: "Trust in checkable specifics was the thing that kept me reading through the middle sag."
- The routable headers taught even the expert something. The skeptic's only +2-and-reread moment: L7 routing and rate-limiting on Mcp-Method without body parsing, "my gateway team's problem solved."
- The transport-vs-application state framing. Both readers independently elected the same sentence as the piece's quotable core: "the transport stopped hiding that state in connection memory; it moved into explicit application handles." The platform engineer called it "the exact rebuttal I need when someone says 'but our tools are stateful.'"
- The honesty beat. "The Google Developers Blog write-up still uses Redis for exactly that" earned the skeptic's rarest compliment: "the anti-hype sentence most vendor posts refuse to write."

## 2. The reading

**The skim gate passed, with a warning.** The scanner committed on the reproducible curl failure demo: "that is engineering, not marketing." But from the skim alone it logged the strike that both full reads later confirmed: no auth story anywhere in the scanner view, and "capabilities in every request" hand-waves where identity lives.

**The platform engineer ran +2 for most of the piece** and was taking notes for Thursday throughout ("affinity fights your autoscaler" stolen verbatim; the GitHub precedent banked as the centerpiece). The dips: chunk 9 at the deploy section ("min-instances=0 is table stakes... --allow-unauthenticated on a GitHub-connected MCP server is a demo-only footgun and the piece hasn't said so"), and the jolt at chunk 17: "the links section quietly reveals the spec is a Release Candidate and the Python/TS SDKs are betas. The whole body spoke in past-tense certainty. Burying RC status in a link label feels like the one genuinely misleading structural choice in the piece." Their pitch downgraded on the spot from "migrate now" to "prototype in staging, wait for GA," and their final note: "the timeline slide is mine to write, not theirs."

**The infra skeptic finished at "worth the read, barely."** Held through the problem sections at +1/0 ("teaching me my own incident history"), peaked at chunks 7 and 8, went -1 through sections 3 and 4 ("this is where the piece stops teaching and starts demoing... ~40% of the words taught a production MCP operator nothing"), and was pulled back by section 5. Their scorecard on the three standing questions: auth never addressed and made worse by the copy-paste --allow-unauthenticated; server-push answered by omission ("progress notifications, sampling, elicitation are simply gone from this story, replaced by 'poll every 2 seconds,' and the piece doesn't acknowledge that as a trade-off"); migration only gestured at ("no dual-version window guidance, no 'what if my client doesn't speak 2026-07-28'"). They also flagged the repro's edges: the "search" tool with arg "q" doesn't match github-mcp-server's real tool names, and "set its session generator to undefined" "is not a real instruction for any server framework I run."

Where the two disagree is itself information: the section that armed the sympathetic reader (the deploy walkthrough) is the section that lost the expert. The piece currently serves the reader who hasn't done this yet and taxes the one who has.

## 3. What survived

- **Sayback vs intended gist: survived, both readers, nearly verbatim.** Both recalls reproduced the full mechanism from memory: SEPs, _meta, task handles, scale-to-zero. The thesis transmits.
- **What stuck:** the architecture sentences ("the transport stopped hiding that state..."), the operational one-liners ("affinity fights your autoscaler," "keep idle pods warm so an HTTP header doesn't get lost," "do not raise your HTTP timeout"), the names and numbers (SEP-2575/2567, -32001, v1.7.0, pollIntervalMs), and the closing frame "making the tool layer boring."
- **Peak:** the skeptic's was chunk 7 (headers and _meta mechanics); the sympathetic reader's was the transport-vs-application framing. Both peaks are in the middle spec sections, not the hook and not the deploy demo.
- **Ending:** survived, but as a jolt. Both recalls independently kept the footer's RC/beta revelation as the ending's emotional content, one recording the reader as "genuinely misled."
- **Center of gravity:** both recalls, unprompted, said the same thing: the piece is most alive in chunks 7-8 and 13-14, the spec mechanics and the state boundary. The title and framing sell serverless deployment; the life is in the spec explainer. The deploy and ADK sections are the flattest stretch for both readers.
- **One action: achieved, with homework.** Sympathetic: assign the 3-step staging test tomorrow, verify the SEPs tonight, rewrite the headline to "delete the session cluster, keep a small task store." Skeptic: drop the wire example in the team channel, tell the gateway team about Mcp-Method routing, share with the caveat "ignore the auth hygiene," and comment about the auth hole under the post.

## 4. The person behind it

The person these sentences imply is a working infrastructure engineer with production scar tissue: the failure enumeration (rerouted, restarted, scaled-to-zero mid-prompt) matches real postmortems, the shell has the \r strip "everyone learns the hard way," and the state-boundary section thinks in architecture rather than benefits. Both readers explicitly registered that person and trusted them.

The seams, where a second voice holds the pen: "scale up to infinity" in the title ("marketing talk," logged in chunk 1), the section-template chrome that thins late ("the scaffolding shows once the content thins"), "from the builder trenches" (the skeptic's only wince), and above all the tense: the body speaks in shipped-fact past tense while the footer's own link labels say Release Candidate and beta. That mismatch is not a word problem; it is the implied author's confidence writing checks the links don't cash, and both readers caught it independently at the same chunk.

The trust ledger (signals.py) agrees: 20 checkable number-bearing tokens and 107 named entities is a heavily funded ledger for 1,570 words, with burstiness 0.62 and portable share 0.34 concentrated in the connective tissue rather than the claims. Epistemically marked share is 0.03: almost nothing in the piece is hedged, which reads as authority while the SEP claims hold and reads as the misleading tense once the RC label surfaces. Admissions counter read zero; my own read confirms the piece does pay one real admission (the Redis-survives concession) but none about its own maturity timeline, which is exactly where one was owed.

## 5. Questions, then opinions by permission

Neutral questions:

1. When a reader finishes this, what do you want them to believe they can do this quarter, as opposed to when the spec goes final?
2. Where do you want the auth story to live: in this piece, or in a linked follow-up the deploy blocks point to?
3. Who is the primary reader of sections 3 and 4: the engineer who has already fought sessions, or the one who hasn't deployed MCP yet? Both readers located the piece's life elsewhere.

I have opinions about where the RC/beta disclosure belongs, about the two --allow-unauthenticated blocks, about naming the polling trade-off, and about what to do with the deploy section's word budget. Want them?

---

*Process appendix: skim gate, both 18-chunk timed reads, and both recall quizzes ran as fresh subagents through the skill's scripts; the orchestrator was contaminated (the author pasted the draft) and therefore performed no reading step itself. The skeptic reader was re-dispatched once after a transient API failure; its replacement started from a clean session. Raw sessions: `skeptic/`, `sympathetic/` under the run directory.*
