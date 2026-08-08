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
  manuscript: rough-idea rounds run the working loop and a draft first,
  manuscript-input rounds start at the panel; every round ends with
  manuscript → streamline → high-level check. The count `ROUNDS` (1–10) is
  chosen at the start of round 1.
- **review report · cross-judgement · rebuttal · ranking** — panel records.

## Pipeline
working loop → draft → review panel → manuscript → streamline → high-level
check → deliverable + decisions (the high-level check is normal-mode only;
fast rounds deliver after streamlining). The review panel is either
**normal mode**
(the 5-agent panel, phases A–F) or **fast mode** (a 2-agent A/B loop) — the
user picks the mode at the start of every round ("Round mode" below). A
**manuscript input** — a complete
document, or the previous round's output — skips the working loop and the
draft and enters the review panel directly: shorter rounds. With `ROUNDS`
set at round 1, the harness runs that many rounds before stopping; each
round appends its subgoals to question.md, records its time cost, and
produces the next manuscript version. Full detail in `reference/`;
paste-ready prompts in `modules/prompts.md`.

## Project start
The **very first question** is a choice: configure the exterior reviewer X
(`X_PROVIDER` / `X_MODEL` / `X_ACCESS`, `modules/providers.md` — provider API
env var or the local Codex CLI; default kimi/k2.7/api), or declare **no X** —
the panel then runs internal A1 + A2 + A3 with explicit roles. Then the
output form (PDF / LaTeX / Markdown / HTML), then the **round count**
(1–10; recommend ≤ 10 — the accumulated record grows each round and beyond
that can exceed the context window), then the run envelope (`RUN_LENGTH`),
then the **round-1 mode** — **normal mode** (the 5-agent panel) or **fast
mode** (the 2-agent loop; "Round mode" below) — then the dossier. Ask these
before any work starts; a no-X panel records `X unavailable — reduced
diversity` and a confidence downgrade. The mode is asked again at the start
of every later round.

## Input → question.md
At round 1, convert the input document — whatever its format (`.tex`, `.md`,
`.pdf`, …) — into `research/question.md`; the dossier locks `Q v1` from it.
At the end of every round, append a dated `Subgoals (round N)` section: the
subgoals obtained that round (open threads, ranking suggestions,
high-level-check targets). The next round opens by reading question.md.

## Round timing
Every round — **normal and fast alike** — is timed by the orchestrator, and
the timestamps appear to the user as the round runs and at its end; timings
are never reconstructed after the fact. At round start, announce
`Round N started <ISO timestamp>` and write it in the dossier **before
spawning any agent**. At each phase boundary, record — and show — the
phase's start/end timestamps in the Round timing table (loop / draft /
panel / manuscript / streamline / check; the normal-mode panel phases A–F
are timed too; fast rounds have no check phase). At round end, compute the
elapsed time from the recorded timestamps, fill the table, and **show the
round time to the user in every mode**: a visible line in the closing
deliverable/decision-list message, e.g. `Round 1: 08:55 → 09:05, elapsed
~10 min (draft 1m · attack 3m · manuscript 3m · streamline 3m)`, always
including the total and, when available, the phase breakdown. Timestamps
are wall-clock times the orchestrator reads at the moment of the event
(e.g. `date`), recorded in a fixed format.

## Working loop
LOAD dossier + question.md → ATTACK one toolkit move (`modules/toolkit.md`) → RECORD
(tried / broke / implies / next) → UPDATE → NEXT. Write as you go. If stuck:
stuck ladder → floor (trial proof with `GAP:` labels, or a worked example).
Never end a session on failure; end on the next action.

## Invariant rules
1. **No agent grades its own homework** — author confidence is never a verdict.
2. **Claims:** `claimed → under-review → accepted | rejected | counterexample`.
   Nothing builds on a non-`accepted` claim; reviewer verdict outranks author
   confidence; ≤ 2 repair rounds per claim per session.
3. **Recording is mandatory.** Dead ends must be recorded; the dossier is the
   memory — append-only, dated, notation-locked, every entry ends with a next
   step.
4. **"Open" is a label, not a verdict.** Intuition = conjecture (recorded,
   then attacked); first test of truth is examples; check literature before
   "impossible".
5. **Everything is versioned.** Every draft, report, manuscript, change list,
   and update to the problem statement carries a version (`v1`, `v2`, …);
   cite the version you build on.
6. **Delegate the tedious; stay high-level.** A massive or tedious task that
   still leaves distance to the goal → spawn a sub-agent, keep thinking
   high-level, record its result in the dossier. Draft sub-agents record
   contradictions honestly; manuscript sub-agents fix contradictions between
   the panel agents.
7. **Agents run in the background.** Every agent Spell spawns — panel
   reviewers, R, M, sub-agents, the high-level check, the auditor — is
   launched in the background and collected when its written artifact is
   ready; the orchestrator never blocks on a spawn.
8. **Measure the panel.** Periodic auditor reads the ledgers (ritualism, gate
   leakage, per-reviewer agreement); canary panels seed known-false claims to
   measure detection. Record what you find.
9. **Artifacts are files; the orchestrator guarantees them.** Every agent
   that produces an artifact — panel reviewers, R, M, sub-agents, the
   high-level check, the auditor — is spawned with an explicit output path,
   must write its artifact there, and must confirm the write in its final
   message. If the agent's environment is read-only or the write fails (a
   read-only sub-agent type, a sandboxed exterior `codex exec`), it must
   instead include the complete artifact text in its final message; the
   orchestrator persists that text verbatim at the assigned path and marks
   the record `recovered from agent output`. The orchestrator checks file
   existence after every agent completes and never starts the next phase on
   a missing artifact.
10. **The chosen mode binds the round.** Round mode (normal/fast) is recorded
    in the dossier at round start, before any agent spawns. A fast round runs
    exactly two agents — A and B — and never spawns the 5-agent panel (A1,
    A2, X/A3, R, M) or runs phases A–F; a normal round runs the full panel.
    If the user chose fast mode, spawning the panel is a protocol violation.

## Review panel — 5 agents, phases A–F (normal mode)
A1 counterexample-hunter + A2 step-validator (internal) + **X** exterior
independent (variables `X_PROVIDER` / `X_MODEL` / `X_ACCESS`; access =
provider API env var or the local Codex CLI) — or internal **A3**
architecture-critic when there is no X + R ranking + M manuscript.

A review → B exchange → C cross-review → D rebuttal → E ranking + close → F
manuscript (follows the ranking; additions flagged `[new in manuscript]`,
disagreements `[ranking deviation]`; must state how it improves the draft).
All panel agents run in the background — A1 + A2 + X/A3 in parallel — their
written artifacts collected in phase order (each agent is spawned with its
artifact path in its prompt; see Invariant rule 9 for agents that cannot
write). In Phase F, M also writes the
**change list** for its manuscript (`changelog-vN.md`): changes vs the
previous version (or vs the input for v1), each with a one-line context and
an importance flag; important changes highlighted at the top.
X unavailable → run with A1/A2 and record `X unavailable — reduced diversity`.

After the panel, a **streamline agent (S)** (fresh context, in the
background) streamlines the manuscript: extracts the core ideas, simplifies
the proof, cuts redundancy — without changing mathematical content
(substance-touching edits are flagged `[streamlined — check]` or left as
suggestions). S appends its simplifications to the change list (context:
streamlining); the high-level check runs on the streamlined manuscript.

## Round mode — normal vs fast (ask at the start of every round)
At the start of **every round — round 1 included** — ask the user: **normal
mode** (the 5-agent panel, phases A–F, above) or **fast mode** (the 2-agent
loop below)? Record the choice in the dossier (`mode: normal | fast` in the
Panel & check ledger) **before any agent spawns**, and follow it for the
whole round.

**In a fast round, the 5-agent panel is NOT run.** Phases A–F do not exist in
a fast round: do not spawn A1, A2, X/A3, R, or M, do not write their prompts,
and do not collect their artifacts. The panel section above applies to normal
mode only. A fast round runs exactly two agents:
- Round 1 — **A** writes the draft (a manuscript input skips the draft); **B**
  attacks it in the background; when
  B's attack arrives, A rebuts and writes the manuscript + change list.
- Rounds N ≥ 2 — **A** attacks the received manuscript; **B** attacks A's
  attack; A rebuts and writes the next manuscript + change list.
The round then continues with the streamline step. **Fast mode skips the
high-level check** — it is the normal-mode gate; a fast round delivers
after streamlining. Fast mode trades review independence for speed — the
author attacks its own artifact, no exterior reviewer, no ranking, no
high-level gate — so every fast round is marked `fast mode` in the ledger
and its outputs carry a confidence downgrade.

## High-level check
Independent gate before delivery: **novelty** (`novel` / `extension` /
`known`) + **sufficiency** (`within reach` / `plausible` / `beyond reach`)
→ `pass` (deliver) / `conditional` (back to loop with targets) / `fail`
(reformulate or park). Strategic gate, not a certificate. **Normal mode
only — fast rounds skip the check and deliver after streamlining.** The
check agent must write its report to a file and confirm the write; a
read-only check agent delivers the report text in its final message and the
orchestrator persists it verbatim (Invariant rule 9).

## Layout
`core.md` (this file — always load) · `modules/` (on-demand) · `reference/`
(authoritative full rules). Registered as a user-scope Skill (`spell`) — the
skill loads this core at 0 tokens until invoked.

Artifacts live in `<project root>/research/`; the dossier and `question.md`
stay at the top. Once a kind — drafts, manuscripts, reports, or change lists —
has **more than 2 versions**, move it into its own folder (`research/drafts/`
· `research/manuscripts/` · `research/reports/` · `research/changelogs/`):
all its versions move there, and new ones are written into the folder.
