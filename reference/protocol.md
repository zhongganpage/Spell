# Spell — Working Protocol

The operating ruleset for a Spell run: how the working sessions behave, how
progress is recorded, and how a rough idea becomes a manuscript.

This file synthesizes two source protocols — the *Persistence Protocol for AI
Agents on Open Mathematical Problems* and the *Independent Verification
Protocol for AI-Agent Claims* — and adapts them to Spell's pipeline
(rough idea or manuscript → idea sprint → tiered review → surgical
manuscript (streamline folded into M) → high-level check → delivery; see
`README.md`). Where this file is silent,
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
7. **Opinions rank; only demonstrated counterexamples kill.** "Known",
   "beyond reach", "misdirected" are opinions — they may demote and attach
   revival triggers, never terminate. The user may always park a thread by
   their own decision; the no-kill rule binds the protocol's mechanisms, not
   the user.
8. **Deterministic checks beat model checks for mechanical error classes.**
   Units, citations, brackets, numerical exact-case reproductions are
   linter/script territory, not reviewer territory.
9. **Round type selects pipeline weight.** A screening round, a negative
   round, a repair round, and a manuscript-bound round do not all run the
   5-agent panel.
10. **The loop must converge monotonically.** Artifact size non-growing,
    open-condition count non-increasing, per-claim verification depth
    non-decreasing. Violation of any of the three is a protocol alarm.
11. **The user steers at verdicts.** `conditional` / `fail` /
    negative-outcome routing is a user decision point (repair / park /
    re-scope), never an automatic continue.
12. **Budget is conserved.** The savings from tier selection fund the new
    phases (sprint, linter, promoter, negative-value assessment); total
    per-round agent budget does not grow.

## 2. The pipeline

```
rough idea / manuscript (input document)
   │  (startup: exterior reviewer, output form, rounds, run envelope,
   │   dossier, input → question.md; per round: idea sprint, then a tier
   │   proposed from the round type — screening | fast | normal)
   ▼
[manuscript input?] ──────────────yes──────────────┐
   │ no                                            │
   ▼                                               │
working loop (+ idea sprint) ────┐                 │  (exploration, §4–§6)
   │                             │                 │
   ▼ (run may end here:          │   conditional:  │
       progress report)          │   user decides  │
draft ───────────────────────────┤   repair / park │
   │                             │   / re-scope    │
   ▼                             │                 │
hygiene linter ◄─────────────────┴─────────────────┘
   │  (deterministic; before any review phase and before
   │   delivery, all tiers)
   ▼
tier (from the round type; user may override; recorded, binds the round)
   │  screening: claim-reviewer format on claims nominated for
   │             verification; no manuscript
   │  fast:      2-agent A/B loop; confidence downgrade
   │  normal:    5-agent panel A–F + promoter
   ▼
manuscript + change list (surgical; streamline folded into M)
   │
   ▼
high-level check (normal mode only — skipped in fast rounds; on negative
   │   rounds replaced by the negative-value assessment, which runs in every
   │   tier; non-terminal routing — fail/misdirected demote + revival
   │   triggers, never auto-park)
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
**manuscript**; (4) at the high-level check, with its verdict; or (5) after
a negative round, with a **negative-value assessment** (§2, "Negative
rounds"). Every run
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

**Round timing.** Every round — in every tier — is timed
automatically by the orchestrator; the timestamps appear in the run's
visible flow and in the dossier, and are never reconstructed or backfilled
after the fact. Checkpoints:
- **Round start** — before any agent spawns, announce and write `Round N
  started <ISO 8601 timestamp>` in the dossier.
- **Phase boundaries** — record and display a timestamp when each phase
  starts and ends (working loop / idea sprint / draft / hygiene linter /
  panel / manuscript / high-level check — check is normal-mode only; on
  negative rounds it is replaced by the negative-value assessment; in
  normal mode the panel phases A–F are timed as well) in the dossier's
  Round timing table.
- **Round end** — compute each phase's elapsed time and the round total
  from the recorded timestamps, fill the Round timing table, report
  total + phase breakdown in the round's deliverable and decision list,
  and **show the round time to the user at the end of the round in every
  tier**: a visible line in the closing message, e.g. `Round 1: 08:55 →
  09:09, elapsed ~14 min (draft 2m · attack 4m · manuscript 5m ·
  linter 1m · check 2m)` (fast rounds show the same line without the
  check phase), always including the total and, when available, the phase
  breakdown.

Timestamps are wall-clock times the orchestrator reads at the moment of the
event (e.g. it runs `date`), in a fixed format.

**Round type and tier.** Every round has a **round type** — what it is for —
and the type selects the **tier**, the pipeline weight, before any agent
spawns. Types: `exploratory` (open attack on an idea or thread), `negative`
(rule-out assessment), `repair-scoped` (fixes only the listed conditions of
a conditional verdict), `manuscript-bound` (produces/delivers a manuscript).
Default tier per type: exploratory → **screening**; negative → **fast**;
repair-scoped → the **cheapest tier that can check the listed conditions**
(normal only when a listed condition is load-bearing); manuscript-bound →
**normal**. The orchestrator proposes a tier from the round type at round
start; the user may override; the choice is recorded in the dossier before
any agent spawns and binds the whole round.

- **screening** — the claim-reviewer format (`modules/prompts.md` §1) on
  every claim nominated for the verification pipeline (§8); no panel, no
  manuscript.
- **fast** — the 2-agent A/B attack/rebut loop instead of the panel; no
  exterior reviewer, no ranking, no high-level check. Round 1: A writes the
  draft, B attacks it in the background, then A rebuts and writes the
  manuscript + change list. Rounds ≥ 2: A attacks the received manuscript,
  B attacks A's attack, then A rebuts and writes the next manuscript +
  change list. The round ends with the hygiene linter; the streamline step
  is folded into M. Fast mode trades review independence for speed — the
  author attacks its own artifact — so every fast round is marked
  `fast mode` in the panel ledger and its outputs carry a confidence
  downgrade.
- **normal** — the 5-agent panel (phases A–F), with the **promoter**
  alongside (`review-panel.md`, "The promoter"); required only for
  load-bearing or manuscript-bound claims.

**Repair-scoped rounds.** A `conditional` verdict is always a user decision
point: **repair** (scope frozen — fix the listed conditions only, no new
conjectures, re-check only those at the end; the artifact is *not*
re-derived), **park** (a user decision, never automatic; fragments + revival
triggers recorded), or **re-scope**. Repair rounds run the cheapest tier
that can check the listed conditions; the repair agent **resumes** the
panel/author agent (§9). No `conditional` verdict ever auto-continues into a
full new round.

**The idea sprint.** The working loop opens with the **idea sprint**, a
parallel phase **hard-capped at 30 minutes wall-clock**, enforced by the
orchestrator (sprint start recorded; collection cut off at the mark; a late
artifact is discarded and recorded `sprint overrun — discarded`): 3–5
**explorer agents** (flash models fine) — specialize/edge-case miner,
reformulation hunter (M5), analogy/transfer agent (reads the obstructions
register and the literature map; evaluates revival triggers), wildcard —
plus a **recombination agent** pairing unrelated dossier entries (two
reformulations; an obstruction + a partial result; an idea + a technique
that worked elsewhere). Each candidate returns with the *cheapest
discriminating test* where one exists — the field is optional, and dead
ends and partial leads count. The sprint **feeds the GAP-owner; it does not open new
target queues**; its surviving candidates become the **sprint backlog** —
the top of the round's attack queue (§4) — settled by the loop this round,
not parked in the register. All candidates — survivors and discards — enter the
**wild-ideas register** with revival triggers (§3), which is capped at
**≤ 15 active** wild ideas; the rest are archived and re-checked only when a
trigger fires or the idea-yield ranking promotes them.

**The hygiene linter.** A **deterministic** mechanical pass by one cheap
agent/script, run **before any review phase and before delivery in all
tiers**: (1) every displayed equation checked for dimensional/normalization
consistency (factor-2, brackets, per-edge vs per-volume, tilt factors);
(2) every citation checked against fetched records (locators present, not
corrupted); (3) every bracket/range/constant checked against the
manuscript's own tables and the notation lock. It is a linter/script, not a
reviewer: nothing it flags is a verdict, but nothing reaches a review phase
or delivery before it has run.

**Negative rounds.** A negative round's deliverable is a
**negative-value assessment** — did we truly rule out a direction, with
evidence (exact cases, fetched literature, certified computation), or did
the tool just fail? It is produced by a **dedicated cheap agent that runs
in all tiers**; it **replaces** the high-level check for negative outcomes
(it is not a form of it), and it **assesses**, never certifies, the evidence
quality of the rule-out. Verdicts: `ruled-out-with-evidence` |
`tool-failure`. A rule-out with assessed evidence is a first-class
deliverable, recorded in the wild-ideas register under the fragments rule
(§3).

**Surgical manuscript.** M patches only the sections the record changed
(diff-scoped) instead of rewriting the full document; the streamline step
folds into M in the same pass — there is no standalone S — with
simplifications that would touch substance flagged `[streamlined — check]`
or left as suggestions with the original step intact; ledgers grow as
appendices rather than being rewritten. Artifact size is monotone
non-growing per round; a violation is a protocol alarm (`review-panel.md`,
Phase F).

A run is finished when it has produced its deliverable — a draft, a
manuscript, a progress report, a **negative-value assessment**, or a parked
state with a decision list — and
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
| **Wild-ideas register** | `speculative` ideas — conjectures, reformulations, analogies, technique transfers not yet claims; exempt from the ledger and panel attack; ≤ 15 active, the rest archived with revival triggers; terminal-verdict fragments land here (§2, §8). |
| **Literature map** | Sources read: theorem, reference, what it leaves open. |
| **Knowledge State** | Populated-and-maintained-at-session-end living summary; the navigation index a fresh session reads first: conjectures registry, obstructions register, champion draft pointer, dependency backlinks (§3). |
| **Attempts log** | Append-only, dated (§4.3). The heart of the protocol. |
| **Claims & verification ledger** | Every claim's lifecycle (§8), including formalization status. |
| **Panel & check ledger** | Each round's run: round type, tier (screening/fast/normal), panel roles (A1, A2, X/A3, R, M — or fast A+B), whether X ran, panel verdict, high-level check verdict (or negative-value assessment on negative rounds), routing. |
| **Round timing** | Per-round start/end timestamps and elapsed time, with a phase breakdown. |
| **Portfolio & threads** | Ranked open threads with a fixed budget split (70% champion / 20% runners-up / 10% wild), re-ranked each round by the idea-yield metric; GAP-owner threads hold the champion share (§6). |
| **Open threads / next steps** | The attack queue. Every entry ends with a concrete next step. |

Rules that make the dossier work: append-only (never rewrite history); dated;
notation-locked; every entry ends with a next step; a fresh session reads the
dossier in full before anything else. Once the dossier exceeds ~30 KB it
splits by section into `claims-ledger.md`, `attempts-log.md`, and
`version-inventory.md`; the top-level dossier stays as the navigation index
(Knowledge State), question.md stays at the top, and the split files keep
the record append-only without concurrent-edit conflicts.

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
context window. It is the dossier's **navigation index** — populated and
maintained every session, never a leftover template — and the first thing a
fresh session reads (§4.1).

**The wild-ideas register.** The dossier section holding `speculative`
ideas — conjectures, reformulations, analogies, technique transfers that are
not yet claims. Entries are exempt from the verification ledger and from
panel attack and leave only by nomination, to `promising` (§8). The register
is **bounded**: ≤ 15 *active* wild ideas; the rest are archived with their
revival triggers and re-checked only when a trigger fires or the idea-yield
ranking promotes them. Every terminal verdict deposits its fragments here —
the maximal true subcase, the obstruction, the closest technique — with
revival triggers (the fragments rule, §8). **Revival triggers** are evaluated
by the auditor at each run and by the sprint's analogy/transfer agent at
each round.

**The portfolio.** Open threads are a ranked population with a fixed budget
split — **70% champion / 20% runners-up / 10% wild** — re-ranked each round
by the **idea-yield metric** (accepted claims + promoted ideas + new
subgoals per unit cost per thread, computed from the ledgers). GAP-owner
threads hold the champion share until the ladder floor (§6).

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

**The round close.** Every round closes with a **single atomic write**: the
round-timing table, the dated subgoals (appended to question.md), the
decision list, and the ledger rows the round changed, generated in one pass
at the round-end phase boundary from the fixed template
(`modules/dossier-template.md`, "Round close"). The user-facing closing
message is derived from that record, never written as a separate ritual —
the "deliverable + decision list" contract (§2) is one write, not five.

## 4. The exploration loop

Every session — and every hour of a long session — runs this loop:

```
1. LOAD     read the dossier; pick the top open thread/next step
2. ATTACK   apply exactly one move from the Transformation Toolkit (§5)
3. RECORD   append the outcome to the attempts log (2–5 lines)
4. UPDATE   adjust status, examples, or reformulations if warranted
5. NEXT     write the next concrete step
```

**The sprint backlog.** When the round ran an idea sprint, its surviving
candidates (each carrying its cheapest discriminating test) form the
**sprint backlog** — the top of the round's attack queue — and the loop
draws from it before free-form toolkit moves: a candidate is settled by
running its discriminating test. The backlog yields to a thread already
mid-flight (a fresh attack never interrupts one in progress); at most 3
backlog draws per round, after which the loop is free; unsettled candidates
keep their revival triggers in the wild-ideas register. Backlog candidates
remain `speculative` until nominated — the backlog is a working queue, not a
promotion (§8).
```

### 4.1 Session start
1. Read the dossier in full — Knowledge State first — and question.md in
   full.
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

### 5.1 Writers are planners; delegation is mandatory for them

**Writers** — the working-loop author (drafts) and the manuscript agent M —
are **planners**: before executing, they decompose the artifact into
concrete tasks (computations, literature sweeps, mechanical case checks,
routine sub-proofs, writing blocks), state the plan, and distribute the
work. Massive or tedious tasks that still leave distance to the goal
**must** go to background sub-agents; small tasks the writer does itself.
A draft or manuscript whose work was not planned and distributed is a
protocol violation. **Reviewers and other pipeline agents** — the panel
reviewers (A1, A2, X/A3), R, the promoter, the high-level check — are
**not** planners: they stay single-shot and do not delegate; the provisions
below apply to writers' sub-agents only.

When the next attack consists of a massive or tedious task — a long
computation, a mechanical case check, a literature sweep, a routine sub-proof
— and completing it still leaves a distance to the ultimate goal, the writer
**delegates it to a sub-agent** instead of sinking its own run into the
details:

1. **Spawn a sub-agent in the background — explicitly.** Open a fresh
   context/session with the harness's explicit background/async spawn — in
   an `Agent`-tool harness (Kimi Code, Claude Code): `run_in_background=true`
   — never the blocking foreground default, and hand it a precise,
   self-contained written task: what to compute, search, or prove; the
   definitions it may use; the exact deliverable format; and the dossier
   entry where the result is recorded. Nothing is transmitted except the
   written task, and nothing comes back except the written result.
2. **Keep thinking high-level.** The delegating agent does not wait idly: it
   continues the high-level line — next moves, reformulations, the shape of
   the artifact — while the sub-agent works.
3. **Sub-agents never grade, never vote.** A delegated result is evidence
   recorded in the dossier — it is not a verification verdict and does not
   bypass the claims ledger (§8). Verification still happens in independent
   sessions with fresh contexts (§8, §11).
4. **Integrate — mandatory, not optional.** Before the artifact is finished,
   the writer: (a) collects every planned sub-agent's written result — a
   missing result is re-spawned or recorded as an open item, never silently
   dropped; (b) reconciles conflicts between sub-agent results and against
   the record — drafts record contradictions honestly, M verifies / repairs /
   decides which side is right; (c) makes every sub-agent outcome that
   appears in the artifact traceable to its delegation (dossier entry); (d)
   writes the artifact only after integration — a result arriving while the
   writer is mid-write triggers a revision pass, not a silent
   incorporation.

**In drafts (working loop).** The author is a planner: it plans the draft's
work in advance, distributes it (mandatory, §5.1), integrates the results
before the draft is finished, and writes the draft so it stays high-level:
the draft records the plan, the delegations, and the integration, not the
mechanics — what was planned, what was delegated, what came back, how it
was reconciled, and what it implies. The details live in the dossier's
delegated-task entries
(`modules/dossier-template.md`, "Delegated tasks"); the draft cites them by
reference. If a sub-agent's result shows the high-level idea is wrong — a
counterexample, a failed case, a violated lemma — the draft reports the
contradiction honestly: the direction, what the detail killed, and what
still stands. A draft that hides a delegated contradiction is worse than
one that reports failure (R3: dead ends that are written down are work).

**In manuscripts (Phase F).** The manuscript agent is a planner: it plans
the manuscript's work in advance (verify disputed claims, re-run
computations, repair gaps, streamline sections, writing blocks), distributes
it (mandatory, §5.1), and its sub-agents carry an extra mandate: they
**not only record** the contradictions between the panel agents — the
review reports, cross-judgements, and rebuttals — they **attempt to fix
them**. For each contradiction a sub-agent verifies the disputed point,
repairs the gap, or determines which side is right, and hands the settled
version back for the manuscript. What cannot be fixed is recorded honestly
as an open item, with the reason (`review-panel.md`, Phase F). Integration
is mandatory: M collects every sub-agent's written result before the
manuscript is finished, reconciles conflicts, and cites each outcome to
its delegation. The change list records the plan, the delegations, and the
integration.

**The formalization anchor.** At least one load-bearing lemma per project is
delegated for Lean 4/mathlib formalization (an M12 delegation,
`modules/prompts.md` §6) — the only check the next round's gate cannot
overturn. Self-contained packages (e.g., a provable d=1 subproblem) are the
default first target. The delegation is bounded by the run envelope like any
M12 task; if no lemma is delegated, the reason is recorded in the dossier
(§8's `formalized`/`unformalized` flags and the delivery note).

## 6. The stuck ladder

A thread is stalled when two consecutive sessions on it produced no progress.
Escalate down the ladder in `modules/toolkit.md` ("The stuck ladder") *in
order*; you may not skip a rung. Only after its final rung may a thread be
marked `stalled` — with the reason and a one-line "resume here" — and a new
thread opened. A stalled thread is a storage state, not a verdict; the
problem itself never becomes abandoned.

**GAP-owner discipline.** One thread owns one GAP across multiple rounds
until the stuck-ladder floor is reached: the owner attacks it from different
angles — each angle recorded in the attempts log — and targets change only
at a floor verdict. The portfolio reserves the champion share for the
GAP-owner (§3). A GAP is attacked from ≥ 2 angles across rounds before it
may be abandoned.

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
speculative → promising → claimed → under-review → accepted | rejected | counterexample
```

- `speculative` — a wild idea, not yet a claim: exempt from the verification
  ledger and from panel attack; leaves the register only by nomination, to
  `promising` (§3, "The wild-ideas register").
- `promising` — nominated for development: may be built upon *heuristically*
  in exploration, every use labeled `heuristic use of <claim vN>`; never a
  premise in any delivered artifact; may appear in drafts, labeled; never in
  user-facing reporting of established results; not subject to review until
  nominated as `claimed`.
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
- **Screening before the panel.** Every claim **nominated for the
  verification pipeline** — entering `claimed`/`under-review` — runs the
  prompts.md §1 claim-reviewer format (the screening tier, §2) *before* any
  panel decision. Only screening-passing claims — or load-bearing ones
  explicitly nominated — reach the panel. A claim that fails screening
  returns to the loop as a repair task; the ≤ 2-repair rule applies.
  `speculative` and `promising` ideas are exempt — they are not yet claims
  in the pipeline.
- **Depth escalation.** Per-claim verification depth is non-decreasing
  across rounds: `screening → panel → formalization`. A claim that survives
  round N is not re-checked at the same depth in round N+1; the ledger's
  `depth` column tracks claim-level depth. A gate verdict is *descriptive
  routing* — to the next depth or to the user — never a certificate.
  Artifact hygiene (the hygiene linter, §2) is the separate axis, run every
  round in every tier.
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
  claims), per-reviewer agreement rates, and **premature kills** — ideas
  `rejected`/`fail`ed/parked that later proved right (a later round, the
  literature, or the user). A single premature kill is a protocol alarm;
  the rate distinguishes **opinion-kills** (verdict-based; a process defect)
  from **evidence-kills** (counterexample-based; correct behavior). The
  auditor also evaluates revival triggers at each run and computes the
  idea-yield metric (§3). A **canary gate** runs inside every normal-tier
  panel: the review batch is seeded with one known-false claim plus one
  planted step-error (both excluded from the real record and the
  manuscript), and the round's manuscript may not be delivered unless the
  panel catches the claim (≥ 80%) and the step-error (100%, with the step
  cited). A miss blocks delivery and is classified in the panel ledger as
  an opinion-kill (verdict-based; protocol alarm) or evidence-kill
  (counterexample-based; correct behavior); the running canary rate is
  recorded in the delivery note. Results are recorded in the dossier and
  fed back as standing panel instructions.
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
reviewers include the exterior agent X when available — written surgically
with the streamline step folded into M (§2), and gated by the high-level
check (`high-level-check.md`). A panel verdict or a high-level
check `pass` is independent review — much stronger than self-checking, but
never a formal proof, and it never overrides a rejected claim in the ledger.

**Post-delivery grounding check.** After delivery, during the user's
decision point (off the round's critical path), one cheap background agent
runs two audits: (1) **novelty spot-check** — a sample of the delivered
`novel`/`extension` verdicts (one in three) is re-run through M11 against
the literature map and the known-vs-negation check; false-`novel` /
false-`known` events are recorded so the error rates accumulate in the
ledger; (2) **numerics audit** — the register's scripts are verified to
exist, agree, and be written by a role other than the theory agent, with
run logs attached (the numerics artifacts are a delivery prerequisite; the
audit verifies it, it does not re-run proofs). Results are recorded in the
dossier and read by the next round.

## 9. Agent mechanics

- **Write as you go.** Record each move immediately after making it. When the
  context window is close to its limit, the attempts-log entry is the
  highest-priority write; then stop cleanly.
- **Agents run in the background — explicitly.** Every agent the run spawns —
  panel reviewers, R, M, sub-agents, the high-level check, the auditor — is
  launched with the harness's explicit background/async spawn — in an
  `Agent`-tool harness (Kimi Code, Claude Code): `run_in_background=true`;
  elsewhere the equivalent async/detached mode — never the blocking
  foreground default, and collected when its written artifact is ready; the
  orchestrator never blocks on a spawn: it launches, keeps doing its own
  high-level work, and integrates the artifact when it arrives. Phase
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
- **Live timestamps.** Phase timestamps are recorded at the moment — the
  orchestrator runs `date` — never reconstructed from notification times or
  backfilled after the fact; a violation is a protocol alarm (§2, "Round
  timing").
- **Codex pipeline.** When the exterior agent runs through the Codex CLI
  (`codex exec`), point it at a writable output dir — or use standardized
  markers in the prompt — so its artifact is written there instead of being
  re-read from 50–100 KB of terminal output; the default is kimi/k2.7/api
  (no sandbox) unless codex is required (§10).
- **Agent resume for repairs.** A repair resumes the panel/author agent that
  produced the artifact — keeping its context — instead of re-reading
  everything fresh (§2, "Repair-scoped rounds").

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
   A1+A2+A3 roles, confidence downgraded` (§8). At startup the harness also
   verifies that `X_MODEL` is not the same provider family as the internal
   harness; if it is — or X is unavailable — the run is auto-labeled
   `reduced diversity` with a confidence downgrade (one-line check,
   non-negotiable).
2. **Ask the user the output form** — PDF, LaTeX, Markdown, or HTML — and
   record the answer (`definition.md`, "Output form").
3. **Fix the round count.** Ask the user how many rounds to run — `ROUNDS`,
   an integer 1–10; recommend ≤ 10 (the accumulated record grows each round
   and can exceed the context window beyond that; `definition.md`,
   "Rounds"). Record it in the dossier.
4. **Fix the run envelope.** `RUN_LENGTH`, chosen at project start. Record it
   in the dossier.
5. **Propose the round tier.** At the start of every round the orchestrator
   proposes a tier from the round type — exploratory → screening; negative →
   fast; repair-scoped → the cheapest tier that can check the listed
   conditions (normal only when a listed condition is load-bearing);
   manuscript-bound → normal. The user may override; the choice is recorded
   in the dossier and binds the round (§2). Re-proposed at the start of
   every round.
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
