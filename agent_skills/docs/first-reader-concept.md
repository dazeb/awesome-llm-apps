# first-reader: a skill that reads like a human instead of auditing like a machine

Research synthesis and skill concept, 26 Aug 2026, branch `evaluate-scandinavian-design-skill`.
Method modeled on ericzakariasson/scandinavian-design: rules as the residue of specific corrections, checks that failed twice become scripts.

Status: BUILT. The skill lives at `agent_skills/first-reader/`, with its
deterministic tests, behavioral evals, fixtures, and development ledger at
`agent_skills/evals/first-reader/`. Round-1 and round-2 experimental
results are in the ledger; this document remains the design rationale and
research record.

## The question

After running any of the ten anti-slop skills, slop still gets through, and the final gate is still a human reading the piece, feeling something is off, and rewriting it in their own language. What is that human doing, and can it be a skill?

## What the research says

Three tracks: a mechanical dissection of all ten anti-slop skills (full SKILL.md text retrieved for each), the cognitive and social psychology of reading, and how professional editors and workshop methods actually review.

### 1. The ten skills automate the heuristic that is known not to work

The landscape is two root documents (Wikipedia's "Signs of AI writing" and stop-slop) forked ten ways, converging on the same ~30 bans: delve, tapestry, em dashes, rule of three, "not X but Y", fake-profound kickers. Across ~150KB of instruction text, the only two agents ever modeled are the **detector** (burstiness, perplexity) and the **author** (voice samples, defendability). The reader is invoked rhetorically ("trust readers") but never given attention, priors, memory, or a quit point.

The psychology literature lands directly on this. Jakesch, Hancock and Naaman (PNAS 2023, N=4,600) showed the cues laypeople use to spot AI text are **wrong**, which is why AI text can score "more human than human." Russell, Karpinska and Iyyer (2025) found the people who detect AI text near-perfectly (a five-person majority vote misclassified 1 of 300 articles, beating commercial detectors and surviving humanizer tools) rely on something else: **originality, specificity, whether the text ever commits to anything, whether confidence varies across claims**. Word-blacklist skills automate the flawed lay heuristic. The accurate detectors judge what the text risks, not which words it uses.

Two skills even delete the most memorable lines on principle ("Cut quotables. If it sounds like a pull-quote, rewrite it"), optimizing not-sounding-AI directly against retention. And the "add soul" recipes are themselves standardizing: three separate skills reach for the same 2am-debugging anecdote. The humanity injection is becoming a new formula.

### 2. What a human reader actually is

The reading science converges on one model: **a forager building a gist model under time pressure, running a trust evaluation of the implied author in parallel.**

- **Foraging.** People read at most ~20-28% of the words on a page (Nielsen, 45k pageviews). Scanning patterns: F-pattern on walls of text, layer-cake across headings, spotted across numbers and bold. Full line-by-line reading happens only after the reader *decides* the piece earned it. ~55% of visits are under 15 seconds; median scroll depth is ~50% (Chartbeat). High-stakes readers are stoppers, not weighers: slush readers reject on paragraph one, VCs average 3:44 on a deck, recruiters 7.4 seconds on a resume. One reason to stop is enough, and an early credibility hit taxes everything after it.
- **Gist, not sentences.** Readers keep a situation model, not a transcript (Kintsch). Verbatim memory collapses within ~80 syllables (Sachs 1967); gist persists (fuzzy trace theory). What survives: the beginning and the end (serial position), the single most intense moment and the final beat (peak-end), and the distinctive isolate in uniform text (von Restorff). **Uniformly polished text has no isolates, so nothing sticks.** That is the memory signature of slop: it reads fine and leaves nothing.
- **The contract.** Relevance theory (Sperber and Wilson): every sentence carries a presumption that its cognitive effect will justify its processing cost. Violations don't register as neutral padding; they trigger inference about the writer (Grice): padding reads as time-wasting, uniform hedging as not-knowing, restating as condescension. Fluency also transfers: hard-to-process prose makes the *author* seem less intelligent (Oppenheimer 2006).
- **The parallel trust channel.** Reading is a parasocial encounter with an implied author (Horton and Wohl; Booth). Credibility is built by costly signals: checkable specifics, admissions against interest (Walster 1966), commitments that could be wrong, confidence that varies by claim. Generic praise and safe hedges are free, hence worthless. Real-experience accounts carry more sensory and contextual detail than fabricated ones, and readers implicitly use detail density as a truth cue. The MIT uncanny-valley work found imperfection and vulnerability *increase* likability; frictionless fluency triggers unease. "Text that emerges from nowhere, addressed to no one, with no stake in its claims" is the fundamental tell, and it is invisible at the word level.

### 3. What a professional reviewer actually does

- **Altitude discipline.** Developmental, then line, then copy, never mixed, because you don't polish sentences in a chapter you're about to cut, and because each pass needs a different reading posture. A real editor refuses to copyedit a structurally broken draft.
- **Feedback is testimony, not verdict.** Peter Elbow's "movies of the reader's mind": narrate, in order and in past tense, what happened in you while reading: expectations formed and broken, where you got lost, where you were won back. The reader cannot be wrong about what they experienced; the writer stays free to decide it doesn't matter. George Saunders runs a P/N needle in his head and logs its position sentence by sentence. Maxwell Perkins wrote to Fitzgerald in exactly this register: "the reader's eyes can never quite focus upon him," and Fitzgerald could act on it instantly.
- **Right about where, wrong about how.** The workshop constant: when readers flag the same place, the place is real, even though every reader's prescription is wrong. So: diagnose place and effect, hold fixes loosely and separately (Liz Lerman's Critical Response Process: meaning first, then questions, then opinions only by permission).
- **The ear detects the missing person, not the banned word.** Editors spot ghost-written and committee text by seams: the implied person changing mid-document, mechanical cleanliness from a writer known to be rough, "volunteer sentences" that no individual chose (Klinkenborg), connective tissue without metabolism.

## The gap, stated plainly

Every existing skill operates on **properties of the text**. The thing you feel when you catch surviving slop is a property of the **experience**: the needle drifting to N, the sentence that cost more than it paid, the paragraph you realize you skimmed, the moment the implied author stopped being a person, the fact that you remember nothing from the middle. No skill occupies that territory. It is also the territory the accurate human AI-detectors operate in.

## The skill: `first-reader`

Not an eleventh deslopper. The reader's chair. It never rewrites; it produces a reading.

### Why an LLM can't just be told "read like a human"

An LLM sees the whole text at once, with perfect memory and infinite patience: three properties no human reader has. Prompting "imagine you're a busy reader" produces performed impatience on top of a god's-eye read. The skill's job is to **mechanically remove those three superpowers**, the way scandinavian-design's scripts removed the model's ability to lie to itself about screenshots.

### Mechanism: five instruments

**1. Cast the reader (script + intake).** Not "the audience" in the abstract. A named persona with priors (what they already know and believe), a situation (phone, feed, inbox, slush pile), a patience budget from the empirical genre data (feed ~15s, recruiter ~7s, memo reader ~4 min, committed subscriber ~full read), and zero cost to quitting. Default: two casts per review, a sympathetic reader and a skeptical gatekeeper (Elbow's believing and doubting games, run as separate labeled passes).

**2. The skim gate (`skim` script, deterministic).** Extract exactly what a scanner fixates on: headings, first ~2 words of each line's opening, first sentences, bold, numbers, links, plus a mobile paragraph-height estimate. Feed *only this view* to a fresh agent and ask: what is this piece about, and do you commit? This is the layer-cake test. Most pieces die here and no anti-slop skill can even represent that.

**3. The timed read (`feed` script, the core).** A driver reveals the text one paragraph at a time to a reader agent with **no lookahead**. At each step the reader logs: what I expect next, needle position (P/N with a reason), effort paid vs effect gained, and whether my cast persona quits here. The output is the movies-of-the-mind transcript: a chronological log with the exact sentence where attention died, where trust broke, where the piece won the reader back, and the predicted stop-point per persona. Restoring the arrow of time is what turns analysis into reading.

**4. The forgetting test (`recall` script).** A separate agent reads the piece once, then, **with the text removed from context**, answers: retell it in one sentence (sayback), quote anything you can (pointing; the von Restorff isolates), what was the peak, how did it end. Diff the retained gist against the author's intended thesis. This is a real experiment, not roleplay: gist that doesn't survive one context boundary won't survive a commute. Also computes Elbow's center of gravity: where the piece came alive vs what it claims to be about.

**5. The trust ledger (`signals` script + judgment).** Construct the implied author ("the person these sentences imply is...") and flag seams where that person changes. Inventory costly signals vs free ones: checkable specifics (numbers, names, dates that could be wrong), admissions against interest, falsifiable commitments, epistemic variance (does confidence differ across claims, or is everything asserted at the same temperature?). A text that pays only free signals reads as marketing or machine, and readers treat those identically.

### The report: Lerman order, testimony register

1. **Sayback and meaning.** What this piece is, as received, and what landed. First, always, because it tells the writer what not to break.
2. **The reading.** The transcript, quoted to the sentence: "At 'X', I expected Y and got Z. The needle went N here and never recovered. My skeptical cast quit at this line; my sympathetic cast finished but skimmed section 3."
3. **What survived.** The recall results: retained gist vs intended thesis, the quotables, the peak, the ending.
4. **The person behind it.** Implied author, seams, trust ledger.
5. **Neutral questions, then opinions by permission.** Prescriptions last, labeled as opinions, declinable.

Altitude rule: if the skim gate or the developmental read fails badly, the skill says so and stops. No line notes on a piece whose structure is dead. And it never rewrites: "putting it in your language" belongs to the writer or to their voice skill (e.g. x-article-writing). Where is the reviewer's job; how is the writer's.

### Guardrails

- **The needle is allowed to stay positive.** An LLM judge always finds something; a first reader sometimes just gets pulled through. "I read it twice and have only pointing to offer" is a valid, reportable outcome, and the eval enforces it with control pieces.
- No scores. A reading, not a rubric. The only numbers are empirical (stop-point, recall content, signal counts).
- Testimony phrasing is enforced: "here is where I stopped believing you," never "this is bad."
- The banned-word layer is out of scope on purpose. Run no-ai-slop first if you want it; this is the gate after.

### Eval (per the repo bar)

- **Adversarial set:** pieces that pass all ten anti-slop skills clean but are hollow (the exact residue Shubham still catches by hand). first-reader must produce a broken reading: no gist retained, flat needle, no costly signals.
- **Control set:** distinctive human writing full of surface "tells" (em dashes, triads, quotables used well). first-reader must produce an engaged reading and must not manufacture complaints. This is the piece every humanizer fails.
- **Ground-truth correlation where it exists:** published pieces with real engagement data; does the predicted stop-point track scroll depth or drop-off? Does the recall test's quotable match what people actually quoted or screenshotted?
- **Reproducibility:** same piece, same cast, three runs; the flagged *places* should agree even when phrasing differs (the workshop invariant: readers agree on where).
- Stress roles per case (stress-hook, stress-middle, stress-trust, stress-voice-seam) plus the control, scandinavian-design style.

### Why this can win

- The tell-catalog ground is saturated: one Wikipedia page forked ten ways. This is the entirely unoccupied layer, and it is the layer the demonstrably accurate human detectors actually use.
- It has real scripts doing real work (exposure gating, context-boundary recall, signal inventory), not prose the model already knows.
- It composes instead of competing: no-ai-slop cleans the surface, first-reader judges the experience, the writer's voice skill does the rewrite. A pipeline, with this as the final gate: which is exactly the role the human currently plays.

### Sources (selected)

Nielsen/NNg reading and scanning studies; Chartbeat engaged-time data; Pirolli and Card, information foraging; Kintsch, construction-integration; Bransford and Franks 1971; fuzzy trace theory; Kahneman peak-end; von Restorff 1933; Sperber and Wilson, Relevance; Grice 1975; Oppenheimer 2006; Jakesch, Hancock, Naaman, PNAS 2023; Russell, Karpinska, Iyyer 2025 (arXiv:2501.15654); Clark et al., ACL 2021; MIT textual uncanny valley (Kishnani 2025); Horton and Wohl 1956; Walster et al. 1966; Zahavi, costly signaling; Berger and Milkman 2012. Editorial: Elbow, Writing Without Teachers / Writing With Power / Sharing and Responding; Saunders, A Swim in a Pond in the Rain; Lerman, Critical Response Process; Perkins' Gatsby letters; McPhee, Draft No. 4; Zinsser; Klinkenborg; Le Guin, Steering the Craft; Gottlieb, Paris Review Art of Editing No. 1; Milford/Clarion workshop method; DocSend pitch-deck data; Ladders eye-tracking study.
