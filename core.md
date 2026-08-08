# Spell — Core

Adversarial multi-agent proof protocol for mathematical theorems. **Contract:**
rough idea or manuscript in → finer manuscript out. No Lean/Coq verification —
the output
is reviewed prose, never a machine-checked proof.
**Run envelope:** `RUN_LENGTH` chosen at project start, bounding agent work
per run; every run ends with a deliverable + decision list; the dossier
carries state across runs; the user decides between runs. Never run
autonomously across days.
**Rounds:** `ROUNDS` — how many rounds to run (1–10, asked at the start of
round 1; recommend ≤ 10 — the dossier, question.md, and panel records grow
every round, and beyond ~10 the accumulated record can exceed the context
window). The harness runs that many rounds before stopping.

## Vocabulary
- **draft** — one agent's single-time report, versioned (`v1`, `v2`, …);
  high-level by design — tedious work is delegated to sub-agents, only its
  outcomes appear; never delivered.
- **manuscript** — post-panel report; the deliverable (output form chosen at
  project start: PDF / LaTeX / Markdown / HTML); versioned, names the draft
  version it improves on.
- **progress report** — run summary (tried / stands / ruled out) + decision
  list; versioned.
- **change list** — per manuscript version: what changed vs the previous
  version (or vs the input artifact for v1), each change with a one-line
  context and an importance flag; important changes highlighted.
- **question.md** — the canonical statement file, converted from the input
  document at project start; subgoals appended after each round.
- **round** — one full cycle from the input artifact to the next delivered
  artifact (manuscript + change list, negative-value assessment, or progress
  report): exploratory/manuscript-bound rounds run the working loop (with the
  idea sprint) and a draft first, manuscript-input rounds start at the hygiene
  linter; negative and repair-scoped rounds are shorter; every round ends with
  the hygiene linter. The count `ROUNDS` (1–10) is chosen at the start of
  round 1.
- **round type** — what the round is for; selects the **tier**:
  `exploratory` (default tier **screening**), `negative` (rule-out
  assessment; default tier **fast**; delivers a **negative-value
  assessment**), `repair-scoped` (fixes only the listed conditions of a
  `conditional` verdict; scope frozen; user-gated), `manuscript-bound`
  (delivers a manuscript; tier **normal**).
- **tier** — pipeline weight chosen at round start from the round type
  (orchestrator proposes; user may override; recorded in the dossier; binds
  the round): **screening** (claim-reviewer format on every claim nominated
  for the verification pipeline; no manuscript) / **fast** (2-agent A/B loop;
  skips panel + high-level check; confidence downgrade) / **normal**
  (5-agent panel + promoter, phases A–F + high-level check; required only for
  load-bearing or manuscript-bound claims).
- **negative-value assessment** — the deliverable of a negative round, in
  every tier: was the direction truly ruled out (exact cases, fetched
  literature, verified computation) or did the tool just fail? Produced by a
  dedicated cheap agent; assesses, never certifies; replaces the high-level
  check for negative outcomes; recorded in the wild-ideas register under the
  fragments rule. Verdicts: `ruled-out-with-evidence` | `tool-failure`.
- **wild-ideas register** — dossier section for **speculative** ideas
  (conjectures, reformulations, analogies, technique transfers not yet
  claims): exempt from the verification ledger and panel attack; leaves only
  by nomination; bounded (≤ 15 active; the rest archived with revival
  triggers).
- **promising** — claim state between `speculative` and `claimed`; may be
  built upon heuristically in exploration, every use labeled `heuristic use
  of <claim vN>`; never a premise in any delivered artifact or in user-facing
  reporting of established results.
- **review report · cross-judgement · rebuttal · ranking** — panel records.

## Pipeline
working loop (+ **idea sprint** at round start) → draft → **hygiene linter**
→ **tier** (screening | fast | normal panel with promoter) → **manuscript**
(surgical — streamline folded into M) → **high-level check** (non-terminal
routing) → deliverable + decisions. The tier is proposed from the round type
at the start of every round; the user may override ("Round tier" below).
Negative rounds deliver a **negative-value assessment** in every tier;
repair-scoped rounds are scope-frozen and user-gated. A **manuscript input**
— a complete document, or the previous round's output — skips the working
loop, the idea sprint, and the draft, and enters the pipeline at the hygiene
linter: shorter rounds. With `ROUNDS` set at round 1, the harness runs that
many rounds before stopping; each round appends its subgoals to question.md,
records its time cost, and produces the next artifact version. Full detail in
`reference/`; paste-ready prompts in `modules/prompts.md`.

## Project start
The **very first question** is a choice: configure the exterior reviewer X
(`X_PROVIDER` / `X_MODEL` / `X_ACCESS`, `modules/providers.md` — provider API
env var or the local Codex CLI; default kimi/k2.7/api), or declare **no X** —
the panel then runs internal A1 + A2 + A3 with explicit roles. Then the
output form (PDF / LaTeX / Markdown / HTML), then the **round count**
(1–10; recommend ≤ 10 — the accumulated record grows each round and beyond
that can exceed the context window), then the run envelope (`RUN_LENGTH`),
then the **round-1 tier** — proposed from the round type (screening | fast |
normal; user may override; "Round tier" below) — then the dossier. Ask these
before any work starts; a no-X panel records `X unavailable — reduced
diversity` and a confidence downgrade. The tier is proposed again at the
start of every later round.

## Input → question.md
At round 1, convert the input document — whatever its format (`.tex`, `.md`,
`.pdf`, …) — into `research/question.md`; the dossier locks `Q v1` from it.
At the end of every round, append a dated `Subgoals (round N)` section: the
subgoals obtained that round (open threads, ranking suggestions,
high-level-check targets). The next round opens by reading question.md.

## Round timing
Every round — **every tier alike** — is timed by the orchestrator, and
the timestamps appear to the user as the round runs and at its end; timings
are never reconstructed after the fact. At round start, announce
`Round N started <ISO timestamp>` and write it in the dossier **before
spawning any agent**. At each phase boundary, record — and show — the
phase's start/end timestamps in the Round timing table (loop / sprint /
draft / linter / panel / manuscript / check; the normal-tier panel phases
A–F are timed too; screening and fast rounds have no check phase — negative
rounds run the negative-value assessment instead). At round end, compute the
elapsed time from the recorded timestamps, fill the table, and **show the
round time to the user in every tier**: a visible line in the closing
deliverable/decision-list message, e.g. `Round 1: 08:55 → 09:05, elapsed
~10 min (draft 1m · attack 3m · manuscript 3m · linter 1m)`, always
including the total and, when available, the phase breakdown. Timestamps
are wall-clock times the orchestrator reads at the moment of the event
(e.g. `date`), recorded in a fixed format. The round close is a **single
atomic write**: the timing table + dated subgoals (appended to question.md)
+ decision list + ledger rows, generated in one pass from the template
(`modules/dossier-template.md`, "Round close"); the closing message is
derived from it, never written as a separate ritual.

## Working loop
LOAD dossier + question.md → ATTACK one toolkit move (`modules/toolkit.md`) → RECORD
(tried / broke / implies / next) → UPDATE → NEXT. Write as you go. If stuck:
stuck ladder → floor (trial proof with `GAP:` labels, or a worked example).
Never end a session on failure; end on the next action.

The author is a **planner**: at session start it decomposes the top next
step into concrete tasks (computations, literature sweeps, case checks,
sub-proofs, writing blocks), states the plan, and distributes the work —
massive or tedious tasks go to background sub-agents (M12, §5.1), each
with a precise written task and a dossier entry for its result; small
tasks the author does itself. Integration is mandatory: before the draft
is finished the author collects every sub-agent's artifact (a missing
result is re-run or recorded as an open item, never silently dropped),
reconciles conflicts between results, and makes each outcome traceable to
its delegation. The draft records the plan, the delegations, and the
integration, not the mechanics. A session whose work was not planned,
distributed, and integrated is a protocol violation.

## Idea sprint
At round start, a parallel sprint **hard-capped at 30 minutes wall-clock**:
3–5 explorer agents (specialize / reformulation hunter / analogy-transfer —
which also evaluates revival triggers / wildcard) plus a **recombination
agent** pairing unrelated dossier entries. Each candidate returns with the
cheapest discriminating test where one exists — the field is optional, and
dead ends and partial leads count; survivors and discards alike enter the
**wild-ideas register** with revival triggers. The sprint feeds the
GAP-owner; it does not open new target queues. The orchestrator records the
sprint start, cuts artifact collection off at the 30-minute mark, and
discards any late artifact (recorded `sprint overrun — discarded`).
Surviving candidates become the **sprint backlog** — the top of the round's
attack queue — settled by the working loop this round, not parked in the
register.

## Sprint backlog
The round's sprint survivors, each with its cheapest discriminating test,
ranked at the top of the attack queue. The working loop draws from the
backlog before free-form toolkit moves; at most 3 draws per round, then the
loop is free; the backlog yields to a thread already mid-flight (§4.1);
unsettled candidates keep their revival triggers and stay `speculative`
until nominated — the backlog is a working queue, not a promotion.

## Hygiene linter
A deterministic mechanical pass (one cheap agent/script), run before any
review phase and before delivery in **all tiers**: every displayed equation
checked for dimensional/normalization consistency (factor-2, brackets,
per-edge vs per-volume); every citation checked against fetched records
(locators present, not corrupted); every bracket/range/constant checked
against the manuscript's own tables and the notation lock. Not a reviewer.

## Invariant rules
1. **No agent grades its own homework** — author confidence is never a verdict.
2. **Claims lifecycle:** `speculative → promising → claimed → under-review →
   accepted | rejected | counterexample`. `speculative` ideas live in the
   wild-ideas register — exempt from the verification ledger and panel attack;
   they leave only by nomination. `promising` claims may be built upon
   heuristically in exploration, every use labeled `heuristic use of <claim
   vN>`; they are never premises in any delivered artifact or in user-facing
   reporting of established results. Nothing builds on a non-`accepted` claim;
   reviewer verdict outranks author confidence; ≤ 2 repair rounds per claim
   per session.
3. **Recording is mandatory.** Dead ends must be recorded; the dossier is the
   memory — append-only, dated, notation-locked, every entry ends with a next
   step.
4. **"Open" is a label, not a verdict.** Intuition = conjecture (recorded,
   then attacked); first test of truth is examples; check literature before
   "impossible".
5. **Everything is versioned.** Every draft, report, manuscript, change list,
   and update to the problem statement carries a version (`v1`, `v2`, …);
   cite the version you build on.
6. **Writers are planners and integrators; reviewers are not.** The
   working-loop author and M plan their work in advance — decompose the
   artifact into tasks, state the plan, distribute: massive or tedious
   tasks go to background sub-agents (M12, §5.1), small ones the writer
   does itself — and integrate before writing: every sub-agent result
   collected, conflicts reconciled, each outcome traceable; the artifact
   records the plan, the delegations, and the integration, not the
   mechanics. Draft sub-agents record contradictions honestly; manuscript
   sub-agents fix contradictions between the panel agents. Reviewers
   (panel, high-level check) are not planners: they stay single-shot, no
   planning, no delegation.
7. **Agents run in the background — explicitly.** Every agent Spell spawns —
   panel reviewers, the promoter, R, M, sub-agents, the high-level check,
   the auditor — is launched with the harness's explicit background/async
   spawn — in an `Agent`-tool harness (Kimi Code, Claude Code):
   `run_in_background=true`; elsewhere the equivalent async/detached mode —
   never the blocking foreground default. The orchestrator never blocks on
   a spawn: it launches, keeps doing its own high-level work, and collects
   the written artifact when it is ready.
8. **Measure the panel.** Each run the auditor reads the ledgers (ritualism,
   gate leakage, per-reviewer agreement, **premature kills** — ideas
   rejected/`fail`ed/parked that later proved right: opinion-kills are a
   protocol alarm, evidence-kills are correct behavior), evaluates the
   **revival triggers** on demoted/archived ideas, and computes the
   **idea-yield** metric (accepted claims + promoted ideas + new subgoals per
   unit cost). A **canary gate** runs inside every normal-tier panel: a
   seeded known-false claim plus one planted step-error ride in the review
   batch (both excluded from the real record and the manuscript), and the
   manuscript may not be delivered unless the panel catches the claim
   (≥ 80%) and the step-error (100%, with the step cited); a miss blocks
   delivery and is classified opinion- vs evidence-kill in the ledger; the
   running detection rate is recorded in the delivery note. Record what you
   find.
9. **Artifacts are files; the orchestrator guarantees them.** Every agent
   that produces an artifact — panel reviewers, the promoter, R, M,
   sub-agents, the high-level check, the auditor — is spawned with an
   explicit output path,
   must write its artifact there, and must confirm the write in its final
   message. If the agent's environment is read-only or the write fails (a
   read-only sub-agent type, a sandboxed exterior `codex exec`), it must
   instead include the complete artifact text in its final message; the
   orchestrator persists that text verbatim at the assigned path and marks
   the record `recovered from agent output`. The orchestrator checks file
   existence after every agent completes and never starts the next phase on
   a missing artifact.
10. **The chosen tier binds the round.** The tier (screening | fast | normal)
    is recorded in the dossier at round start, before any agent spawns. A
    screening round runs the claim-reviewer format only; a fast round runs
    exactly two agents — A and B — and never spawns the 5-agent panel (A1,
    A2, X/A3, R, M, P) or runs phases A–F; a normal round runs the full panel
    + promoter. If the user chose a cheaper tier, spawning the panel is a
    protocol violation.

## Review panel — 5 agents + promoter, phases A–F (normal tier)
A1 counterexample-hunter + A2 step-validator (internal) + **X** exterior
independent (variables `X_PROVIDER` / `X_MODEL` / `X_ACCESS`; access =
provider API env var or the local Codex CLI) — or internal **A3**
architecture-critic when there is no X + R ranking + M manuscript + **P**
promoter (fresh context, alongside the panel): pushes the champion idea as
far as it goes — the strongest honest version, the maximal true fragment,
exactly where it breaks. Its outputs enter the ledger as `promising` /
`claimed`; its own work never grades itself (invariant 1).

A review → B exchange → C cross-review → D rebuttal → E ranking + close → F
manuscript (surgical — patches only the sections the record changed; follows
the ranking; additions flagged `[new in manuscript]`, disagreements `[ranking
deviation]`; must state how it improves the draft).
All panel agents run in the background — A1 + A2 + X/A3 in parallel, P
alongside — their
written artifacts collected in phase order (each agent is spawned with its
artifact path in its prompt; see Invariant rule 9 for agents that cannot
write). In Phase F, M also writes the
**change list** for its manuscript (`changelog-vN.md`): changes vs the
previous version (or vs the input for v1), each with a one-line context and
an importance flag; important changes highlighted at the top.
X unavailable → run with A1/A2 and record `X unavailable — reduced diversity`.

**Streamline folds into M.** M streamlines in the same pass — extracts the
core ideas, simplifies the proof, cuts redundancy — without changing
mathematical content (simplifications that would touch substance are flagged
`[streamlined — check]` or left as suggestions with the original step
intact); there is no standalone S phase. Ledgers grow as appendices; artifact
size is monotone non-growing per round (a protocol alarm otherwise); the
high-level check runs on the manuscript M produced.

## Round tier — screening | fast | normal (proposed at round start, user may override)
At the start of **every round — round 1 included** — the orchestrator proposes
the tier from the round type; the user may override. Record the choice in the
dossier (`tier: screening | fast | normal` in the Panel & check ledger)
**before any agent spawns**, and follow it for the whole round.
- **screening** — per-claim review in the claim-reviewer format
  (`modules/prompts.md` §1) on every claim nominated for the verification
  pipeline; default for exploratory rounds; no manuscript is produced.
- **fast** — the 2-agent A/B loop (below); default for negative and
  repair-scoped rounds (repair escalates to normal only when a listed
  condition is load-bearing); skips the panel and the high-level check.
- **normal** — the 5-agent panel + promoter (phases A–F) + high-level check;
  required only for load-bearing or manuscript-bound claims.

A **negative round** delivers a **negative-value assessment** in every tier —
it replaces the high-level check for negative outcomes, it is not a form of
it. A **repair-scoped round** fixes only the listed conditions of a
`conditional` verdict — scope frozen (no new conjectures), re-check only
those conditions, artifact not re-derived — and runs the cheapest tier that
can check them. `conditional` is always a user decision point (repair / park
/ re-scope), never an automatic continue.

**In a screening or fast round, the 5-agent panel is NOT run.** Phases A–F do
not exist in those rounds: do not spawn A1, A2, X/A3, R, M, or P, do not write
their prompts, and do not collect their artifacts. The panel section above
applies to normal tier only. A fast round runs exactly two agents:
- Round 1 — **A** writes the draft (a manuscript input skips the draft); **B**
  attacks it in the background; when
  B's attack arrives, A rebuts and writes the manuscript + change list.
- Rounds N ≥ 2 — **A** attacks the received manuscript; **B** attacks A's
  attack; A rebuts and writes the next manuscript + change list.
**Fast mode skips the high-level check** — it is the normal-tier gate; a fast
round delivers after M. Fast mode trades review independence for speed — the
author attacks its own artifact, no exterior reviewer, no ranking, no
high-level gate — so every fast round is marked `fast mode` in the ledger
and its outputs carry a confidence downgrade.

## High-level check
Independent gate before delivery: **novelty** (`novel` / `extension` /
`known`) + **sufficiency** (`within reach` / `plausible` / `beyond reach`)
→ `pass` (deliver) / `conditional` (user decision: repair / park / re-scope —
never an automatic continue) / `fail` (demote + attach revival triggers).
**Non-terminal routing:** no verdict auto-parks a thread — `fail` and
"misdirected" demote the direction and attach revival triggers ("re-examine
when <event>"); the only terminal state for an idea is a **demonstrated
counterexample with a reproducible computation**, and every terminal verdict
deposits its fragments into the wild-ideas register (fragments rule: maximal
true subcase, obstruction, closest technique). Strategic gate, not a
certificate. **Normal tier only — screening and fast rounds skip the check;
negative rounds replace it with the negative-value assessment.** The check
agent must write its report to a file and confirm the write; a read-only
check agent delivers the report text in its final message and the
orchestrator persists it verbatim (Invariant rule 9).

## Post-delivery check
After delivery, during the user's decision point (off the round's critical
path), one cheap background agent runs two grounding audits: (1) **novelty
spot-check** — a sample of the delivered `novel`/`extension` verdicts (one
in three) is re-run through M11 against the literature map and the
known-vs-negation check; false-`novel` / false-`known` events are recorded
so the error rates accumulate in the ledger; (2) **numerics audit** — the
register's scripts are verified to exist, agree, and be written by a role
other than the theory agent, with run logs attached. The numerics artifacts
are a delivery prerequisite; the audit verifies it, it does not re-run
proofs. Results are recorded in the dossier and read by the next round.

## Layout
`core.md` (this file — always load) · `modules/` (on-demand) · `reference/`
(authoritative full rules). Registered as a user-scope Skill (`spell`) — the
skill loads this core at 0 tokens until invoked.

Artifacts live in `<project root>/research/`; the dossier and `question.md`
stay at the top. Once a kind — drafts, manuscripts, reports, or change lists —
has **more than 2 versions**, move it into its own folder (`research/drafts/`
· `research/manuscripts/` · `research/reports/` · `research/changelogs/`):
all its versions move there, and new ones are written into the folder.
