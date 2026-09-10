# 👁️ first-reader

**Readers before you publish.**

Two simulated readers with lives meet your draft cold, one passage at a
time, and leave their honest comments beside your text: where they
leaned in, where they drifted, where they would have quit, what they
still remembered the next day. A skimmer tells you whether they'd even
open it. Then you can ask any of them follow-up questions. Nobody
rewrites anything; the draft stays yours.

There are at least ten anti-slop and humanizer skills in circulation, and
they share one architecture: a catalog of properties of the *text* (banned
words, sentence shapes, em dashes, rule-of-three). The research says that
layer automates the wrong heuristic: laypeople's word-level cues for AI
text are demonstrably flawed (Jakesch et al., PNAS 2023), while the people
who detect hollow text almost perfectly judge what a piece commits to,
risks, and leaves in memory (Russell et al., 2025). That is an experience
layer, and no skill modeled it. This one does.

first-reader simulates a specific reader encountering your draft cold, and
reports the reading:

- **Skim gate**: `skim.py` extracts exactly what a scanner fixates on
  (headings, first lines, bold, numbers) and a fresh reader decides from
  that view alone whether to commit. For most real readers this gate is
  the whole encounter.
- **Timed read, no lookahead**: `feed.py` reveals the piece one beat at a
  time and refuses the next chunk until the reader logs what just
  happened: needle position, expectation vs. reality, skimming, doubt,
  quitting. Humans read forward in time; this restores the arrow of time
  to a model that otherwise sees everything at once.
- **Recall test**: `recall.py` hands a fresh agent only the reading log,
  never the draft, and quizzes it: retell the piece in one sentence, quote
  what stuck, name the peak, remember the ending. What cannot be
  reconstructed from the log did not survive the read.
- **Trust ledger**: `signals.py` counts costly signals (checkable numbers,
  named entities, admissions against interest) versus free ones (blanket
  hedging, uniform confidence, portable sentences that fit any document).
- **The report**: reader testimony in workshop order: what landed first,
  then the transcript with exact quit points, what survived, the person
  the prose implies. Never scores. Never rewrites: where is the reviewer's
  job, how is the writer's.
- **The page**: `room.py` turns the run into a single page, titled "First read: <your title>": your
  draft with the readers' comments beside every passage, the skimmer's
  verdict, a needle strip you can scrub, quit points drawn as a fold
  line ("S stopped here. They never saw anything below this line."),
  and lenses that show what a skimmer sees, what survived in memory,
  and where the trust signals live.
- **Ask the readers**: `ask.py` lets you consult any reader afterwards;
  they answer in character from their own reading log, never from a
  fresh look at the text.

## Install

```bash
npx skills add https://github.com/Shubhamsaboo/awesome-llm-apps/tree/main/agent_skills/first-reader
```

Or by hand: clone the repo and copy `agent_skills/first-reader` into
`~/.claude/skills/` (Claude Code, personal), into a project's
`.claude/skills/` (that project only), or wherever your agent loads
`SKILL.md` folders from; Cursor, Codex, and friends all work.
Everything inside is Markdown and stdlib Python: offline, no
dependencies, nothing leaves your machine. Then say:

> be my first reader on this draft

## Use

- **"review this"** (with a path, or just paste it): your agent goes
  quiet for a few minutes, then tells you what a friend would after
  reading: whether the skimmer opened it, where each reader leaned in
  or left, what stuck, and one question back to you. Plus a page: your
  draft with the readers' comments beside every passage.
- **Ask the readers.** "ask S what would have convinced her", "ask the
  skimmer what would have made them open it", "ask everyone if they
  noticed the retention example". They answer in character, from what
  they actually experienced, not from a fresh look at the text.
- **Revise in your own editor, then "again".** Same readers, fresh
  minds; the page shows last time's attention strip above this time's,
  so you see whether your revision moved them.
- **"quick read"**: sixty seconds, one skeptical scanner, two sentences.

## Who reads it

Every read uses two readers: one who wants the piece to be good and
one looking for a reason to stop. Where they come from is up to you,
in three tiers of effort.

**1. Say it in the sentence.** "review this, it's for backend
engineers on our company blog" is enough. The skill casts both readers
from the venue and audience you named. Say nothing and it asks one
question (who is this for, where will it run) or infers from the draft
and states the assumption in its verdict.

**2. Describe the people.** The strongest input is a one-line life, not
a job title. Compare:

> an AI engineer

> an AI engineer whose agent's quality check is reading transcripts by
> hand at 6pm

The second has what makes a reader's boredom or interest mean
something: what they already know, where they meet your text, and why
they might care. Give both readers, or just the skeptic ("read it as a
staff engineer who has built three eval harnesses and thinks this is
vendor content").

**3. Save them once per project.** Put `.first-reader/audience.json` next
to your drafts and every read in that folder uses the same two people,
so "S" is the same person next month and "again" compares like with
like:

```json
{
  "venue": "X long-form, AI engineers",
  "readers": {
    "keen": {"initial": "E", "label": "adk engineer",
             "title": "an AI engineer whose quality check is reading transcripts by hand"},
    "skeptic": {"initial": "S", "label": "eval skeptic",
                "title": "a staff engineer who has built eval harnesses and wants a reason to stop"}
  }
}
```

"Read it as X instead" overrides the saved readers for one run.

Two rules hold no matter what you ask for: it never casts you, and it
never casts someone who loves your style. Readers are a test, and a fan
is not one.

## What it looks like in your coding agent

First use, any agent:

```
you:    review drafts/launch-post.md, it's going on our blog for
        backend engineers
agent:  Reading it as your audience would, about 5 minutes. When I'm
        back: where each reader leaned in or left, what stuck with
        them, and a page with their comments beside your draft.
        ...(several minutes; the readers are working)...
agent:  The skimmer moved on at the title. Both readers who opened it
        finished; S nearly left at the setup section ("two paragraphs
        of pitch before anything happens") and came back at your
        outage story. The next day both still had "the outage was a
        for loop"; neither remembered the ending. One question: is
        the outage the piece? Page: <link>
you:    ask S what would have kept her in the setup section
agent:  S: "One real number. You told me the tool was fast; I'd have
        stayed for '40 ms' and left for 'blazing'."
```

What varies by agent, and the skill adapts on its own:

| | Readers | The page |
|---|---|---|
| Claude Code on claude.ai | fresh subagents, sealed off from your draft | published private page |
| Codex, Cursor, OpenClaw, Antigravity, any CLI | fresh subagents where the harness has them, otherwise the agent plays each reader in turn and says so | local HTML file beside the draft, opened for you |

The page is a report, not an app: nothing on it needs a server, so it
behaves identically everywhere. Your questions to the readers travel
the way every instruction to a coding agent does: you say them.

The one honest caveat for single-agent environments: the readers there
are role-passes by the same mind, not fresh minds, so the skill weights
its mechanical checks more heavily and tells you to treat the felt
findings as slightly softer. Everything else, including the scripts
that pace the reading and test what survives in memory, runs the same
everywhere: plain Python, offline.

It composes with, rather than replaces, the writing-side skills. Editor
skills like [addyosmani/clarity](https://github.com/addyosmani/clarity)
work the writer's chair: substance, truth, register, craft. first-reader
works the reader's chair: what actually happens when someone meets the
result cold. Write with an editor, verify with a reader; the two check
each other, and neither can do the other's job.

## From your editor

The skill is a sentence to your agent, so any editor that can run a
shell command can start a read on the file you have open. VS Code, as a
task in `.vscode/tasks.json`:

```json
{"label": "first read this file", "type": "shell",
 "command": "claude \"be my first reader on ${file}\""}
```

Obsidian users can bind the same command with the Shell Commands
plugin; the page opens in your browser either way.

## Evidence

Tested against paired fixtures built as opposites: a piece that passes
every anti-slop ban yet commits to nothing, and a piece full of surface
"tells" (em dashes, triads, a quotable kicker) that is staked and
specific. The simulated skeptic quit the hollow piece at the same chunk
across independent reruns ("the same post I have read a hundred times
under other titles") and read every word of the telly one, checking its
math along the way, without flagging a single em dash. Transcripts,
recall diffs, and the development ledger live in
[`evals/first-reader/`](../evals/first-reader/).

Then run five times on one real essay, same two readers each time,
the author revising between runs from the readers' comments alone. The
skeptic quit at passage 9 on the first draft and finished every draft
after; by the fourth read neither reader logged a negative passage,
and the peak passage and the three lines that survived recall were the
same in all five reads. The round-by-round record is in the ledger.
