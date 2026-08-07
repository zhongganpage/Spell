# Spell — Working Protocol

The operating ruleset for a Spell run: how the working sessions behave, how
progress is recorded, and how a rough idea becomes a manuscript.

This file synthesizes two source protocols — the *Persistence Protocol for AI
Agents on Open Mathematical Problems* and the *Independent Verification
Protocol for AI-Agent Claims* — and adapts them to Spell's pipeline
(rough idea → draft → review panel → manuscript → high-level check →
delivery; see `README.md`). Where this file is silent, the source protocols
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
   default `medium` = 60 min); Spell never operates autonomously across days
   or months.

## 2. The pipeline

```
rough idea
   │  (startup: output form, run envelope, exterior agent, dossier)
   ▼
working loop ───────────────────────────────┐  (exploration, §4–§6)
   │                                        │
   ▼ (run may end here: progress report)    │  fail / conditional
draft (single agent, one time)              │
   │                                        │
   ▼                                        │
review panel → manuscript ──────────────────┤
   │                                        │
   ▼                                        │
high-level check ───────────────────────────┘
   │
   ▼
deliverable + decision list → user → (next run, if the user continues)
```

**The run envelope.** A Spell run is time-boxed: at most **`RUN_LENGTH`** of
agent work per run — `quick` 30 min · `medium` 60 min · `long` 2 h ·
`superlong` 4 h, chosen at project start (default `medium`, 60 min) — after
which the run must produce a deliverable and a decision list, and control
returns to the user. A full rough-idea → manuscript cycle
may span several runs: a run may end (1) after the working loop, with a
**progress report**; (2) after a draft; (3) after the panel, with a
**manuscript**; or (4) at the high-level check, with its verdict. Every run
end is a **user decision point** — continue, redirect, stop, or a new rough
idea. The dossier carries the state across runs.

A run is finished when it has produced its deliverable — a draft, a
manuscript, a progress report, or a parked state with a decision list — and
handed the decision list to the user. Any other exit is a return to the
working loop or a parked thread in the dossier.

## 3. The dossier

One living, append-only file per problem: `<project root>/research/
dossier-<problem>.md`. Sections, in order:

| Section | Contents |
|---|---|
| **Status** | `OPEN — n threads active` · last session date. Never `abandoned`. |
| **Problem statement (locked)** | Precise statement, all definitions, the **chosen output form** (asked at project start), the **run envelope**, and the **exterior agent** configuration. |
| **Notation** | One fixed notation block; all entries must use it. |
| **Examples & computations** | Dated table: case, computed value, pattern observed. |
| **Reformulations & connections** | Numbered forms, each with "useful because…" and a date. |
| **Literature map** | Sources read: theorem, reference, what it leaves open. |
| **Attempts log** | Append-only, dated (§4.3). The heart of the protocol. |
| **Claims & verification ledger** | Every claim's lifecycle (§8). |
| **Panel & check ledger** | Each draft → manuscript run: panel roles (A1, A2, X, R, M), whether X ran, panel verdict, high-level check verdict, routing. |
| **Open threads / next steps** | The attack queue. Every entry ends with a concrete next step. |

Rules that make the dossier work: append-only (never rewrite history); dated;
notation-locked; every entry ends with a next step; a fresh session reads the
dossier in full before anything else.

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
1. Read the dossier in full.
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

When the run budget (`RUN_LENGTH`, default `medium` 60 min) is nearly spent, the run must not start
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

- **M1 — Compute examples.** Smallest cases by hand or code; build a table;
  state the pattern as a conjecture (recorded); then hunt the *smallest
  counterexample* to it.
- **M2 — Extremes and limits.** Degenerate cases: zero, empty set, identity,
  trivial group, $n=0$, $p=\infty$, $\varepsilon \to 0$, $n \to \infty$.
  Extremes expose the mechanism that intermediate cases hide.
- **M3 — Specialize.** Restrict to a subfamily provable with known tools;
  prove it completely — its proof usually reveals the general mechanism.
- **M4 — Generalize.** Strip hypotheses to the minimal setting. The proof
  often falls out of the cleaner statement.
- **M5 — Reformulate.** Translate the problem into a different language:
  fixed-point, optimization, variational, combinatorial, probabilistic,
  graph-theoretic, or generating-function. Change variables, normalize, take
  the dual, complement, contrapositive. Every new form is new search fuel.
- **M6 — Prove partial directions.** The necessary direction; a weaker
  bound; a restricted class. A half-proved statement with an exact boundary
  of where the proof stops is a research result in itself.
- **M7 — Key-lemma hunt.** State the lemma that would solve the problem: "If
  Lemma X, then the theorem follows." Prove X standalone, or find its
  counterexample. The bottleneck is usually the whole problem in miniature.
- **M8 — Extremal principle.** Assume a minimal counterexample / maximal
  element / worst case and derive a contradiction.
- **M9 — Induction and monotonicity.** Is there a natural induction step? A
  monotonicity that makes a limiting argument work?
- **M10 — Computational experiments.** Brute force over small ranges, random
  search for counterexamples, symbolic tests (OEIS, a CAS). Experiments
  inform; they never substitute for a proof.
- **M11 — Literature triangulation.** For every reformulation, search its
  terminology. Find the nearest known theorem and read its proof *fully* —
  adapted, not cited. Check specifically whether the claim (or its negation)
  is already known — this distinguishes "open" from "impossible".

## 6. The stuck ladder

A thread is stalled when two consecutive sessions on it produced no progress.
Escalate down this ladder *in order*; you may not skip a rung:

1. **Untried moves.** Run the toolkit moves you have not yet tried (keep a
   tick list per thread).
2. **Re-read the dossier.** There are usually unused threads and
   half-written entries from earlier sessions.
3. **New reformulation → new search.** Each new form of the problem is a new
   set of literature search terms.
4. **Reduce to a baby case.** Prove a trivial case completely and write it up
   as if for publication.
5. **Split the problem.** Prove *any* provable subproblem and record it as a
   partial result — a durable asset.
6. **The floor — prove something anyway.** If no next step exists even now,
   the session must not end empty-handed: produce a **trial proof** (best
   argument you can write, every gap labelled `GAP:` in the text) or a
   **proof for a semi-explicit example** (a concrete instance worked through
   completely). Trivial is fine and encouraged; the point is engagement. A
   trial proof is not a claim of correctness — record it as
   draft/`claimed` and let verification judge it.
7. **Only now:** mark the thread `stalled` — with the reason and a one-line
   "resume here" — and open a new thread. A stalled thread is a storage
   state, not a verdict. The problem itself never becomes abandoned.

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
- **Diversity is recorded.** Every panel and review row carries the
  reviewer's model/backend and whether it was an internal agent or the
  exterior agent X. If X was unavailable, the row records `X unavailable —
  reduced diversity` and the manuscript or verdict carries the same mark.

**In the Spell pipeline.** Individual claims are verified by single
independent sessions (ready-to-paste reviewer prompt in §11). The manuscript
itself is produced by the review panel (`review-panel.md`) — whose three
reviewers include the exterior agent X when available — and gated by the
high-level check (`high-level-check.md`). A panel verdict or a high-level
check `pass` is independent review — much stronger than self-checking, but
never a formal proof, and it never overrides a rejected claim in the ledger.

## 9. Agent mechanics

- **Write as you go.** Record each move immediately after making it. When the
  context window is close to its limit, the attempts-log entry is the
  highest-priority write; then stop cleanly.
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

## 10. Startup checklist

When a new Spell project begins, in order:

1. **Ask the user the output form** — PDF, LaTeX, Markdown, or HTML — and
   record the answer (`definition.md`, "Output form").
2. **Fix the run envelope.** `RUN_LENGTH` — `quick` 30 min · `medium`
   60 min · `long` 2 h · `superlong` 4 h; default `medium` (60 min). Record it
   in the dossier.
3. **Configure the exterior panel agent.** Ask the user for the three
   variables `X_PROVIDER`, `X_MODEL`, `X_ACCESS` (`review-panel.md`,
   "The exterior agent (X)"). `X_ACCESS` is `api` (key in an environment
   variable or secrets store — never in the dossier or any report) or
   `codex` (the pre-installed Codex CLI, no key needed). Current default:
   `X_PROVIDER=kimi`, `X_MODEL=k2.7`, `X_ACCESS=api` (`MOONSHOT_API_KEY`).
   If the configured access is unavailable at panel start, X is unavailable
   and the run records the reduced diversity (§8).
4. **Create the dossier** (template in §12 below), filling in the locked
   problem statement, notation, output form, run envelope, and exterior
   agent.
5. **Fix the internal record format** (LaTeX for math-heavy projects,
   Markdown otherwise) for panel reports, rebuttals, rankings, and ledger
   entries.
6. Open the first thread and write its first next step.
7. Begin the exploration loop (§4).

The first draft is produced only after the dossier exists: the draft is the
first artifact that leaves the working loop, and it must be able to cite its
thread and move.

## 11. The reviewer prompt (ready to paste)

For verifying individual claims outside the panel, paste this into a second,
fresh session with the claim, definitions, cited results, and proof attached:

```text
Independently review the mathematical claim and proof below. You have not
seen how the author derived them; do not assume the author's reasoning is
sound. Judge only what is written.

A. CLAIM — Is the statement precise, complete in its hypotheses,
   well-defined in its terms, and exactly what it claims to be (not weaker,
   not a different claim)?
B. PROOF — Does every step follow from the results it cites, and only from
   them? Do the cited results' hypotheses hold at each point of use? Is any
   step silently assuming something unproved? Do the quantifiers match the
   claim?
C. BOUNDARY — Check degenerate and edge cases, check the smallest nontrivial
   case, and hunt for a counterexample to the claim or to a single step.

Verdict, with reasons: accepted | rejected (with specific repair targets) |
gaps found (non-blocking notes).

--- CLAIM (exactly as stated) ---
<claim text>

--- DEFINITIONS (exactly) ---
<definitions used>

--- CITED RESULTS (statements only, not proofs) ---
<cited results>

--- PROOF (exactly as written) ---
<proof text>
```

## 12. Template dossier

Copy this file per problem. Fill in the locked section once; the rest grows
by appending.

```markdown
# Dossier: <short problem name>

**Status:** OPEN — 0 threads active | **Last session:** <date>

## Problem statement (locked)

<precise statement, all definitions>

**Output form:** <PDF | LaTeX | Markdown | HTML — asked at project start>

**Run envelope:** RUN_LENGTH=<quick | medium | long | superlong> — 30 min · 60 min · 2 h · 4 h per run (chosen at project start)

**Exterior panel agent:** <provider/model — configured at start; credentials
kept in environment variables or a secrets store, never in this dossier>

**Internal record format:** <LaTeX | Markdown>

## Notation

- <symbol> : <meaning>

## Examples & computations

| date | case | computed value / observation | pattern? |

## Reformulations & connections

- **R1.** <form of the problem> — useful because <reason> (<date>)

## Literature map

| date | source | theorem / claim | hypotheses | gap it leaves |

## Attempts log

### <date> — thread <T#>, move <M#>
tried:   ...
broke:   ...
implies: ...
next:    ...

## Claims & verification ledger

| date | claim | status | reviewer (model) | verdict & reasons | repair targets / notes |

## Panel & check ledger

| date | artifact | panel (A1,A2,X,R,M) | panel verdict | high-level check | routing |

> In the panel cell, record which agents actually ran — e.g. `X:ok` or
> `X:unavailable — reduced diversity` (`protocol.md` §8).

## Open threads / next steps

- **T1** <thread description> — state: active — next: <concrete step>
- **T2** <thread description> — state: stalled <date> — resume: <one line>
```

Example attempts-log entry:

```markdown
### 2026-08-06 — thread T1, move M3 (specialize to n = 2)
tried:   proved the claim for the 2-variable case by explicit expansion.
broke:   the expansion needs a commutation property that fails for n >= 3.
implies: the obstruction is noncommutativity, not size — try commuting
         generators, or a nilpotent family, next.
next:    state and test the "commuting generators" subcase; search
         literature for "commutative case <problem name>".
```
