# Spell — Working Protocol

The operating ruleset for a Spell run: how the working sessions behave, how
progress is recorded, and how a rough idea becomes a manuscript.

This file synthesizes two source protocols — the *Persistence Protocol for AI
Agents on Open Mathematical Problems* and the *Independent Verification
Protocol for AI-Agent Claims* — and adapts them to Spell's pipeline
(rough idea or manuscript → review panel → manuscript → streamline →
high-level check → delivery; see `README.md`). Where this file is silent,
the source protocols
apply. Documents referenced here: `definition.md`, `review-panel.md`,
`high-level-check.md`.

## 1. Core principles

1. **The unit of progress is not the solution; it is an entry in the
   dossier.** A session is successful if and only if it leaves the dossier
   strictly more informative.
2. **No agent grades its own homework.** A claim is *established* only after
   an independent review accepted both the claim and the proof. Author
   confidence is never a verdict.
3. **Recording is mandatory.** An unrecorded dead end is guaranteed to be
   re-attempted; an unrecorded result is lost work.
4. **The dossier is the memory.** Nothing an agent knows lives only in a
   previous context window.
5. **Runs are time-boxed; the user decides between runs.** A run ends with a
   deliverable and a decision list within the run budget (`RUN_LENGTH`,
   chosen at project start); Spell never operates autonomously across days
   or months.
6. **Everything is versioned.** Every draft, progress report, manuscript,
   change list, and update to the problem statement carries a version label
   (`v1`, `v2`, …).
   Nothing is cited, reviewed, or built upon without naming its version; an
   unversioned artifact is not a valid reference.

## 2. The pipeline

```
rough idea / manuscript (input document)
   │  (startup: exterior reviewer, output form, rounds, run envelope,
   │   dossier, input → question.md; each round: normal or fast mode)
   ▼
[manuscript input?] ──────────────yes──────────────┐
   │ no                                            │
   ▼                                               │
working loop ────────────────────┐                 │  (exploration, §4–§6)
   │                             │                 │
   ▼ (run may end here:          │   fail/conditional
       progress report)          │                 │
draft ───────────────────────────┤                 │
   │                             │                 │
   ▼                             │                 │
review panel ◄───────────────────┴─────────────────┘
   │  (normal: A–F · fast: 2-agent attack/rebut loop;
   │   every artifact versioned, change list per manuscript)
   ▼
manuscript + change list
   │
   ▼
streamline (agent S)
   │
   ▼
high-level check (normal mode only — skipped in fast rounds)
   │
   ▼
deliverable + decision list → user → (next run, if the user continues)
```

**Two input modes.** Spell accepts either a **rough idea** (informal,
incomplete — goes through the working loop and a draft first) or a
**manuscript** (a complete document, including a previous round's output —
enters the review panel directly: no working loop, no draft). The manuscript
mode shortens each round; new ideas from the user or the loop can still enter
at any time.

**The input document → question.md.** At round 1 the input document —
whatever its format (`.tex`, `.md`, `.pdf`, …) — is converted into
`<project root>/research/question.md`, the canonical statement file; the
dossier's locked statement (`Q v1`) is derived from it. At the end of every
round, a dated `Subgoals (round N)` section is appended to question.md: the
subgoals obtained that round (open threads, ranking suggestions,
high-level-check targets). The next round opens by reading question.md.

**The run envelope.** A Spell run is time-boxed: at most **`RUN_LENGTH`** of
agent work per run, chosen at project start — after
which the run must produce a deliverable and a decision list, and control
returns to the user. A full rough-idea → manuscript cycle
may span several runs: a run may end (1) after the working loop, with a
**progress report**; (2) after a draft; (3) after the panel, with a
**manuscript**; or (4) at the high-level check, with its verdict. Every run
end is a **user decision point** — continue, redirect, stop, or a new rough
idea. The dossier carries the state across runs.

**The round count.** At the start of round 1 the user is asked **`ROUNDS`** —
how many rounds to run, an integer from 1 to 10 (recommended ≤ 10: the
dossier, question.md, and panel records grow every round, and beyond ~10 the
accumulated record can exceed the harness's context window). The harness then
runs up to `ROUNDS` full cycles — each from the current input artifact to the
next manuscript and its change list — and stops with the deliverable and the
decision list. The user may extend or interrupt at any point; `ROUNDS` is
recorded in the dossier's locked section. The round count operates inside the
run envelope: `RUN_LENGTH` is still enforced at phase boundaries, and if it
runs out mid-cycle the run closes with a progress report and the remaining
rounds resume in a later run.

**Round timing.** Every round — normal and fast alike — is timed
automatically by the orchestrator; the timestamps appear in the run's
visible flow and in the dossier, and are never reconstructed or backfilled
after the fact. Checkpoints:
- **Round start** — before any agent spawns, announce and write `Round N
  started <ISO 8601 timestamp>` in the dossier.
- **Phase boundaries** — record and display a timestamp when each phase
  starts and ends (working loop / draft / panel / manuscript / streamline /
  high-level check — check is normal-mode only; in normal mode the panel
  phases A–F are timed as well) in the dossier's Round timing table.
- **Round end** — compute each phase's elapsed time and the round total
  from the recorded timestamps, fill the Round timing table, report
  total + phase breakdown in the round's deliverable and decision list,
  and **show the round time to the user at the end of the round in every
  mode**: a visible line in the closing message, e.g. `Round 1: 08:55 →
  09:09, elapsed ~14 min (draft 1m · attack 3m · manuscript 3m ·
  streamline 3m · check 4m)` (fast rounds show the same line without the
  check phase), always including the total and, when available, the phase
  breakdown.

Timestamps are wall-clock times the orchestrator reads at the moment of the
event (e.g. it runs `date`), in a fixed format.

**Round mode — normal vs fast.** At the start of every round — round 1
included — ask the user whether to run the round in **normal mode** (the
5-agent panel, phases A–F) or **fast mode** (a 2-agent loop instead of the
panel; `review-panel.md`, "Fast mode"). Record the choice in the dossier
(`mode: normal | fast` in the panel ledger) before any agent spawns; the
mode binds the whole round. **In a fast round the 5-agent panel is not run
at all** — no A1, A2, X/A3, R, or M, no phases A–F; exactly two agents run.
Round 1: A writes the draft, B attacks it
in the background, then A rebuts and writes the manuscript + change list.
Rounds ≥ 2: A attacks the received manuscript, B attacks A's attack, then A
rebuts and writes the next manuscript + change list. The round still ends
with the streamline step; **fast mode skips the high-level check** (the
normal-mode gate) and delivers after streamlining. Fast mode trades review
independence for speed — the author attacks its own artifact, no exterior
reviewer, no ranking, no high-level gate — so every fast round is marked
`fast mode` in the panel ledger and its outputs carry a confidence
downgrade.

**The streamline step.** Between the manuscript and the high-level check, a
**streamline agent (S)** — a fresh context, run in the background — reads the
manuscript and its change list and tries to streamline it: extract the core
ideas, simplify the proof, cut redundancy, without changing the mathematical
content (`review-panel.md`, "The streamline step"). S appends its material
simplifications to the change list (context: streamlining); in normal mode
the high-level check runs on the streamlined manuscript, and in fast mode
the round delivers after streamlining (no check).

A run is finished when it has produced its deliverable — a draft, a
manuscript, a progress report, or a parked state with a decision list — and
handed the decision list to the user. Any other exit is a return to the
working loop or a parked thread in the dossier.

Every delivered manuscript carries a one-page **delivery note**: models per
role and diversity achieved, checks run and their verdicts, claims still
`claimed`/`under-review`, computed vs. opined, fetched vs. remembered, and
formalization status. The note operationalizes "nothing is a certificate" at
the moment of delivery.

## 3. The dossier

One living, append-only file per problem: `<project root>/research/
dossier-<problem>.md`, beside `<project root>/research/question.md` (the
canonical statement, §2). Sections, in order:

| Section | Contents |
|---|---|
| **Status** | `OPEN — n threads active` · last session date. Never `abandoned`. |
| **Problem statement (locked)** | Precise statement, all definitions, the **chosen output form** (asked at project start), the **run envelope**, the **round count**, and the **exterior agent** configuration. |
| **Notation** | One fixed notation block; all entries must use it. |
| **Examples & computations** | Dated table: case, computed value, pattern observed. |
| **Reformulations & connections** | Numbered forms, each with "useful because…" and a date. |
| **Literature map** | Sources read: theorem, reference, what it leaves open. |
| **Knowledge State** | Rewritten-at-session-end living summary: conjectures registry, obstructions register, champion draft pointer, dependency backlinks (§3). |
| **Attempts log** | Append-only, dated (§4.3). The heart of the protocol. |
| **Claims & verification ledger** | Every claim's lifecycle (§8), including formalization status. |
| **Panel & check ledger** | Each round's run: mode (normal/fast), panel roles (A1, A2, X/A3, R, M — or fast A+B), whether X ran, streamline run (S), panel verdict, high-level check verdict, routing. |
| **Round timing** | Per-round start/end timestamps and elapsed time, with a phase breakdown. |
| **Open threads / next steps** | The attack queue. Every entry ends with a concrete next step. |

Rules that make the dossier work: append-only (never rewrite history); dated;
notation-locked; every entry ends with a next step; a fresh session reads the
dossier in full before anything else.

**Artifact versions.** Every draft, progress report, and manuscript carries a
version (`v1`, `v2`, …), incremented each time a new one is produced; review
reports, cross-judgements, rebuttals, rankings, and change lists are versioned
the same way.
Updates to the locked problem statement are versioned too (`Q v1` → `Q v2`,
…), each with a dated note of what changed and why — the statement is
append-only, never silently rewritten. Ledger rows cite the artifact versions
they record (e.g. "draft v2"), the panel reviews a specific draft version, a
manuscript names the draft version it improves on, and anything that builds on
an artifact names its version.

**The Knowledge State.** At every session end the dossier's Knowledge State
section is rewritten (the only rewritten section; the archive below it stays
append-only): a **conjectures registry** (`active` / `supported` / `refuted`
/ `parked-until <trigger>`), an **obstructions register** (each obstruction
with the move that created it), the **champion draft pointer** (best current
artifact + version), and **dependency backlinks** between claims and the
artifacts that build on them. When a claim hits `counterexample`, its
dependents are flagged `affected` and re-verified before reuse. The Knowledge
State keeps the live picture readable even after the archive outgrows a
context window.

**Artifact layout.** Artifacts live in `<project root>/research/`; the dossier
and question.md stay at the top. Once a kind — drafts, manuscripts, reports
(progress reports and panel records), or change lists — has **more than 2
versions**, it moves into its own folder: `research/drafts/`,
`research/manuscripts/`, `research/reports/`, `research/changelogs/`. All
existing versions of that kind move into the folder; later versions are
written directly into it.

**Change lists.** Every manuscript version is accompanied by a **change
list** (`changelog-vN.md`): what changed vs the previous manuscript version
(or vs the artifact it improves on, for v1), each change with a one-line
context (which review finding / ranking item / repair drove it) and an
importance flag (`high` / `medium` / `low`); the `high` changes are repeated
at the top as a highlighted "Key changes" list. The change list is written by
the manuscript agent in Phase F (`review-panel.md`) and its version is
recorded in the panel ledger row.

## 4. The exploration loop

Every session — and every hour of a long session — runs this loop:

```
1. LOAD     read the dossier; pick the top open thread/next step
2. ATTACK   apply exactly one move from the Transformation Toolkit (§5)
3. RECORD   append the outcome to the attempts log (2–5 lines)
4. UPDATE   adjust status, examples, or reformulations if warranted
5. NEXT     write the next concrete step
```

### 4.1 Session start
1. Read the dossier and question.md in full.
2. Restate the problem in your own words and compare with the locked
   statement — this catches definition drift early.
3. Take the top "next step" written by the previous session. Do not start a
   fresh attack from scratch when a thread is already mid-flight.

### 4.2 Session end (the non-negotiable part)
Before ending, in this order:
1. Append the session's entry to the attempts log (write-as-you-go makes
   this a formality).
2. Update the status line and any tables that changed.
3. Write the next step. If you cannot write one, walk down the Stuck Ladder
   (§6) until you can — creating a next step is part of the work.

When the run budget (`RUN_LENGTH`) is nearly spent, the run must not start
new work: it closes the current entry, writes the run's deliverable — a
draft, a manuscript, or a **progress report** (what was tried, what stands,
what was ruled out, and the decision list) — and stops.

### 4.3 Attempts-log entry (five fields, no more, no less)

```
### <date> — thread <T#>, move <M#>
tried:    <exactly what was attempted>
broke:    <where it failed, or "no progress">
implies:  <what this excludes / suggests / leaves open>
next:     <one concrete next action>
```

A "no progress" entry is a good entry. An entry missing `broke` or `next` is
a bad one and must be completed before the session ends.

## 5. The transformation toolkit

When stuck, run through these moves *mechanically*; each move, even a failed
one, is recorded. This is the repertoire that answers "no next move".

The moves M1–M12 are defined in `modules/toolkit.md`; the delegation rule
that M12 invokes is spelled out below (§5.1).

### 5.1 Delegating tedious work

When the next attack consists of a massive or tedious task — a long
computation, a mechanical case check, a literature sweep, a routine sub-proof
— and completing it still leaves a distance to the ultimate goal, the agent
in charge **delegates it to a sub-agent** instead of sinking its own run into
the details:

1. **Spawn a sub-agent in the background.** Open a fresh context/session in
   the background (async/detached mode) with a precise, self-contained
   written task: what to compute, search, or prove; the definitions it may
   use; the exact deliverable format; and the dossier entry where the result
   is recorded. Nothing is transmitted except the written task, and nothing
   comes back except the written result.
2. **Keep thinking high-level.** The delegating agent does not wait idly: it
   continues the high-level line — next moves, reformulations, the shape of
   the artifact — while the sub-agent works, then integrates the result when
   it arrives.
3. **Sub-agents never grade, never vote.** A delegated result is evidence
   recorded in the dossier — it is not a verification verdict and does not
   bypass the claims ledger (§8). Verification still happens in independent
   sessions with fresh contexts (§8, §11).

**In drafts (working loop).** The author delegates so the draft stays
high-level: the draft records the delegation, not the mechanics — what was
delegated, what came back, and what it implies. The details live in the
dossier's delegated-task entries (`modules/dossier-template.md`, "Delegated
tasks"); the draft cites them by reference. If a sub-agent's result shows the
high-level idea is wrong — a counterexample, a failed case, a violated lemma
— the draft reports the contradiction honestly: the direction, what the
detail killed, and what still stands. A draft that hides a delegated
contradiction is worse than one that reports failure (R3: dead ends that are
written down are work).

**In manuscripts (Phase F).** The manuscript agent may delegate tedious work
the same way, and its sub-agents carry an extra mandate: they **not only
record** the contradictions between the panel agents — the review reports,
cross-judgements, and rebuttals — they **attempt to fix them**. For each
contradiction a sub-agent verifies the disputed point, repairs the gap, or
determines which side is right, and hands the settled version back for the
manuscript. What cannot be fixed is recorded honestly as an open item, with
the reason (`review-panel.md`, Phase F).

## 6. The stuck ladder

A thread is stalled when two consecutive sessions on it produced no progress.
Escalate down the ladder in `modules/toolkit.md` ("The stuck ladder") *in
order*; you may not skip a rung. Only after its final rung may a thread be
marked `stalled` — with the reason and a one-line "resume here" — and a new
thread opened. A stalled thread is a storage state, not a verdict; the
problem itself never becomes abandoned.

## 7. Anti-give-up rules

- **R1 — "Open" is a label, not a verdict.** Never conclude a session with
  "cannot be solved", "too hard", or "unknown". Conclude with the dossier's
  status and next step.
- **R2 — No session ends on a failure.** Sessions end on the next action. If
  none exists, create one via the Stuck Ladder.
- **R3 — Dead ends must be recorded.** An unrecorded dead end does not count
  as work and is guaranteed to be re-attempted.
- **R4 — Every reformulation is recorded.** Even ones that go nowhere.
- **R5 — Intuition is a conjecture, not a fact.** State it as a recorded
  conjecture, then attack it or search for its counterexample before building
  on it.
- **R6 — The first test of "is it true?" is examples.** Compute before
  concluding; a wrong belief is expensive.
- **R7 — One new literature source per session** until the literature map
  covers the area's known results and their gaps.
- **R8 — Persistence is accumulation, not repetition.** Two stalled sessions
  on a thread mean switch moves or threads, not try the same attack harder.
- **R9 — "I don't know" is not "impossible."** Before asserting a statement
  is false or hopeless, check the literature and search for a counterexample.
- **R10 — After a breakthrough, exploit it.** Record it, update the tables,
  state what it unlocks as new threads, write the next conjecture. A
  breakthrough without a follow-up is a lost session.
- **R11 — The minimum-output floor.** When no direction remains, the session
  must still not end empty-handed: produce a **trial proof** (every gap
  labelled `GAP:`) or a **proof for a semi-explicit example**, and record it
  as an attempts-log entry and as draft/`claimed` in the verification ledger.

## 8. Verification and the claims ledger

Every claimed theorem, proposition, lemma, corollary, counterexample,
reduction, asserted computation, and nonstandard definition gets an
independent review — the main goal is not special.

**Lifecycle:**

```
claimed → under-review → accepted | rejected | counterexample
```

- `claimed` — the agent believes the result; recorded with the claim text and
  the location of its proof. Nothing may build on it yet.
- `under-review` — handed to an independent session. Still nothing may build
  on it.
- `accepted` — the independent session verified the claim *and* the proof.
  Only now may later work treat it as established.
- `rejected` — the reviewer found a flaw; record the repair targets; the
  claim returns to the author as an ordinary attempts-log repair task, then
  goes back to review.
- `counterexample` — the claim is false as stated. Fix the statement, or
  record the falsity and move on.
- `formalized` / `unformalized` — status flags on an `accepted` claim: it was
  (or was not) delegated (M12) for a Lean 4/mathlib formalization attempt.
  Formalization is optional — the only path to a machine-checked artifact —
  and the ledger and the delivery note always record which of the two holds.

**Rules:**

- A claim in `claimed` or `under-review` is never used as a premise — not by
  later lemmas, not in reporting to the user.
- **The repair loop is bounded:** at most two review rounds per session. A
  claim still rejected after two rounds is parked as `rejected` with its full
  history; a fresh attack starts a new thread.
- **The reviewer's verdict outranks the author's confidence.** The author may
  repair and resubmit; it may not overrule. Contesting a verdict is itself a
  new claim — "the reviewer's objection is mistaken" — sent to an independent
  session with the objection and the reviewer's report attached.
- **A rejection is not a failure entry.** It is a normal, recorded outcome
  (R3: dead ends that are written down are work).
- **Every review round is a ledger row** (date, claim, status, reviewer,
  verdict & reasons, repair targets). History is preserved, never overwritten.
- **Tiered verification.** Code-verifiable claims get a scripted check (an
  M12 sub-agent runs the computation) instead of a full review session;
  claims that only extend accepted claims get delta-only review against the
  accepted base; `floor`-tagged trial artifacts never enter the review queue.
- **The panel is measured.** A periodic auditor (every few runs, or on
  request) reads both ledgers and computes: ritualism (Phase-A `gap` tags
  later withdrawn), gate leakage (`pass` manuscripts accumulating `rejected`
  claims), and per-reviewer agreement rates. Canary panels — a seeded
  known-false claim — measure the detection rate. Results are recorded in
  the dossier and fed back as standing panel instructions.
- **Diversity is recorded.** Every panel and review row carries the
  reviewer's model/backend, its role (A1 counterexample hunter, A2 step
  validator, A3 architecture critic, X exterior, or fast-mode A/B), and
  whether X ran. If X
  was unavailable — or the user chose no X — the row records `X unavailable
  — reduced diversity, A1+A2+A3 roles, confidence downgraded`, and the
  manuscript, verdict, and delivery note carry the same mark.

**In the Spell pipeline.** Individual claims are verified by single
independent sessions (ready-to-paste reviewer prompt in §11). The manuscript
itself is produced by the review panel (`review-panel.md`) — whose three
reviewers include the exterior agent X when available — then streamlined
(§2) and gated by the high-level check (`high-level-check.md`). A panel
verdict or a high-level
check `pass` is independent review — much stronger than self-checking, but
never a formal proof, and it never overrides a rejected claim in the ledger.

## 9. Agent mechanics

- **Write as you go.** Record each move immediately after making it. When the
  context window is close to its limit, the attempts-log entry is the
  highest-priority write; then stop cleanly.
- **Agents run in the background.** Every agent the run spawns — panel
  reviewers, R, M, sub-agents, the high-level check, the auditor — is
  launched in the background (async/detached mode) and collected when its
  written artifact is ready; the orchestrator never blocks on a spawn. Phase
  order is preserved: a phase starts only when its written inputs exist.
- **Artifact delivery contract.** Every spawned agent is given an explicit
  output path in its prompt and must (a) write its artifact there and (b)
  confirm the write in its final message. An agent whose environment cannot
  write — a read-only sub-agent type, or a sandboxed exterior run
  (`codex exec` under `sandbox_mode: read-only`, approval `never`) —
  delivers the complete artifact text in its final message; the orchestrator
  persists it verbatim at the assigned path, records it as
  `recovered from agent output`, and verifies the file exists before the
  next phase starts. Prefer a writable agent type for any agent that must
  produce an artifact.
- **Fresh sessions load the dossier.** A new session reads the dossier in
  full and continues the top next step. It does not re-derive notation,
  re-read all the literature, or re-attempt recorded dead ends.
- **Search failures are recorded too.** "Searched X, found nothing relevant"
  is a valid literature-map entry.
- **Separate "known" from "believed."** Entries citing external results carry
  the source; the agent's own reasoning is marked as such. The verifier must
  be able to tell them apart.
- **Never reuse a stale fact.** An entry is valid evidence only while its
  source and hypotheses still hold.
- **Do not re-derive locked statements.** The problem statement and notation
  live in one locked place; restating them at session start is for
  drift-checking only.
- **Sub-agents are delegations, not verdicts.** A spawned sub-agent completes
  one precise, tedious task and writes its result into the dossier; it does
  not grade the delegating agent's work (§5.1). Verdicts still come only from
  independent review sessions (§8, §11).

## 10. Startup checklist

When a new Spell project begins, in order:

1. **Choose the exterior reviewer — the very first question.** Ask the user
   to choose: (a) **configure X** — the three variables `X_PROVIDER`,
   `X_MODEL`, `X_ACCESS` (`review-panel.md`, "The exterior agent (X)").
   `X_ACCESS` is `api` (key in an environment variable or secrets store —
   never in the dossier or any report) or `codex` (the pre-installed Codex
   CLI, no key needed). Current default: `X_PROVIDER=kimi`, `X_MODEL=k2.7`,
   `X_ACCESS=api` (`MOONSHOT_API_KEY`). Or (b) **declare no X** — the panel
   then runs internal A1 + A2 + A3 with explicit roles. Record the choice in
   the dossier. If X is chosen but unavailable at panel start, the panel
   falls back to A1 + A2 + A3 and records `X unavailable — reduced diversity,
   A1+A2+A3 roles, confidence downgraded` (§8).
2. **Ask the user the output form** — PDF, LaTeX, Markdown, or HTML — and
   record the answer (`definition.md`, "Output form").
3. **Fix the round count.** Ask the user how many rounds to run — `ROUNDS`,
   an integer 1–10; recommend ≤ 10 (the accumulated record grows each round
   and can exceed the context window beyond that; `definition.md`,
   "Rounds"). Record it in the dossier.
4. **Fix the run envelope.** `RUN_LENGTH`, chosen at project start. Record it
   in the dossier.
5. **Fix the round mode.** Ask whether round 1 runs in **normal mode** (the
   5-agent panel, phases A–F) or **fast mode** (the 2-agent loop); record
   `mode: normal | fast` in the dossier. Re-ask at the start of every later
   round.
6. **Create the dossier** (template in §12 below), filling in the locked
   problem statement (`Q v1`), notation, output form, round count, run
   envelope, and the exterior-agent choice (X or none).
7. **Convert the input document into question.md.** Whatever the input is —
   `.tex`, `.md`, `.pdf`, or any other format — turn it into
   `<project root>/research/question.md`, the canonical statement from which
   `Q v1` is locked. If the input is already a manuscript, note in the
   dossier that the run enters the review panel directly (§2).
8. **Fix the internal record format** (LaTeX for math-heavy projects,
   Markdown otherwise) for panel reports, rebuttals, rankings, and ledger
   entries.
9. Open the first thread and write its first next step.
10. Begin the exploration loop (§4) — or the review panel directly, in
    manuscript-input mode.

The first draft is produced only after the dossier exists: the draft is the
first artifact that leaves the working loop, and it must be able to cite its
thread and move. In manuscript-input mode there is no first draft — the first
artifacts are the panel record and the next manuscript.

## 11. The reviewer prompt (ready to paste)

For verifying individual claims outside the panel, paste the claim-reviewer
prompt (`modules/prompts.md` §1) into a second, fresh session with the
claim, definitions, cited results, and proof attached.

## 12. Template dossier

The canonical dossier template — the locked section, ledgers, round timing,
and delivery note — and its example attempts-log entry live in
`modules/dossier-template.md`. Copy it per problem; fill in the locked
section once; the rest grows by appending.
