# 👁️ First Reader

Readers before you publish.

Anti-slop and humanizer skills audit properties of your text: banned words, sentence shapes, em dashes. None of them can tell you the thing that decides whether a piece works, which is what happens to a person reading it. A reader skims the headline and decides in two seconds. A reader gets to passage four, drifts, and leaves without ever seeing your best paragraph. A reader finishes and remembers one line the next day, or none.

This skill puts two simulated readers with lives in front of your draft, cold, one passage at a time, and reports what happened to them: where they leaned in, where they drifted, where they quit, what they still remembered the next morning. A skimmer says whether they would open it at all. Afterwards you can ask any reader a follow-up. Nobody rewrites anything. The draft stays yours.

## Install

```bash
npx skills add https://github.com/Shubhamsaboo/awesome-llm-apps/tree/main/agent_skills/first-reader
```

The [skills CLI](https://skills.sh) installs the folder for compatible
coding agents. You can also copy this directory into an agent's skills
directory. Everything inside is Markdown and stdlib Python: offline, no
dependencies, nothing leaves your machine.

## Use

Point your agent at a draft, or paste it:

> review this, it's a launch post for backend engineers on our blog

The agent goes quiet for a few minutes, then tells you what a friend would
after reading: whether the skimmer opened it, where each reader leaned in
or left, what stuck with them, and one question back to you. It also gives
you a page: your draft with the readers' comments beside every passage,
the skimmer's verdict, and a strip showing attention passage by passage.

Then keep talking to the readers:

- **"ask S what would have kept her past passage 4"**: she answers from
  her own reading log, in her voice, and never proposes a rewrite.
- **"again"** after you revise in your own editor: same readers, fresh
  minds, and the page draws last read's attention strip above this one.
- **"quick read"**: sixty seconds, one skeptical skimmer, two sentences.

Say who the piece is for in the first sentence and the readers are cast
from that. Describe them as people rather than job titles ("a staff
engineer who has built three eval harnesses and wants a reason to stop")
and the readings get sharper. Save them once in `.first-reader/audience.json`
beside your drafts and every read in that folder uses the same two
people, so "S" means the same person next month.

## Why readers instead of rules

- A text scanner can only measure the text. Every word-level rule has
  been shown to be a weak signal of whether writing is hollow, while the
  people who spot hollow writing near-perfectly judge what a piece
  commits to, risks, and leaves in memory. That is an experience, so
  this skill simulates the experience.
- The readers cannot cheat. A served feed reveals one passage at a time
  and refuses to advance faster than a person could read, so a reader
  never sees what is below the line they quit at.
- Memory is measured, not asserted. The next-day quiz is answered from
  the reader's log alone, never from a fresh look at the text.
- The readers stay out of your chair. They say what happened to them and
  what would have had to be true for it to go differently. Every decision
  about the text is yours.

## Verify

```bash
python3 agent_skills/evals/first-reader/test_first_reader.py
```

The eval builds temporary drafts and checks the scanner view, the served
feed (no text on disk, dwell enforcement, per-reader tokens), recall
bundles, the trust ledger, the reader consult, and the page. The
behavior spec lives in
[`agent_skills/evals/first-reader/`](../evals/first-reader/): run each
prompt in `evals.json` in a fresh session with the skill installed and
check the listed expectations; `trigger-cases.json` covers when the skill
must and must not fire. The development ledger there records six rounds
of testing, including five consecutive reads of one real essay.

## Files

```text
first-reader/
|-- SKILL.md
|-- README.md
|-- references/
|   |-- personas.md
|   |-- report.md
|   |-- room.md
|   `-- interview.md
`-- scripts/
    |-- skim.py
    |-- feed.py
    |-- recall.py
    |-- signals.py
    |-- ask.py
    |-- room.py
    `-- room_template.html
```

Part of [awesome-llm-apps](https://github.com/Shubhamsaboo/awesome-llm-apps).
Apache-2.0. Last verified: September 2026.
