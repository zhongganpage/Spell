# Spell — Core

Adversarial multi-agent prover for mathematical theorem proofs. **Contract:**
rough idea in → manuscript out. **Run envelope:** a variable `RUN_LENGTH`
chosen at project start — `quick` (30 min) · `medium` (60 min) · `long` (2 h)
· `superlong` (4 h) — of agent work per run; every run ends with a deliverable
+ a decision list for the user; the dossier carries state across runs; the
user decides between runs. Never run autonomously across days.

## Vocabulary
- **draft** — one agent's single-time report; never delivered.
- **manuscript** — post-panel report; the deliverable (output form chosen at
  project start: PDF / LaTeX / Markdown / HTML).
- **progress report** — run summary (tried / stands / ruled out) + decision list.
- **review report · cross-judgement · rebuttal · ranking** — panel records.

## Pipeline
working loop → draft → review panel → manuscript → high-level check →
deliverable + decisions. Full detail in `reference/`; paste-ready prompts in
`modules/prompts.md`.

## Working loop
LOAD dossier → ATTACK one toolkit move (`modules/toolkit.md`) → RECORD
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

## Review panel — 5 agents, phases A–F
A1, A2 internal + **X** exterior (variables `X_PROVIDER` / `X_MODEL` /
`X_ACCESS`; access = provider API env var or the local Codex CLI) + R
ranking + M manuscript.

A review → B exchange → C cross-review → D rebuttal → E ranking + close → F
manuscript (follows the ranking; additions flagged `[new in manuscript]`,
disagreements `[ranking deviation]`; must state how it improves the draft).
X unavailable → run with A1/A2 and record `X unavailable — reduced diversity`.

## High-level check
Independent gate before delivery: **novelty** (`novel` / `extension` /
`known`) + **sufficiency** (`within reach` / `plausible` / `beyond reach`)
→ `pass` (deliver) / `conditional` (back to loop with targets) / `fail`
(reformulate or park). Strategic gate, not a certificate.

## Layout
`core.md` (this file — always load) · `modules/` (on-demand) · `reference/`
(authoritative full rules). Registered as a user-scope Skill (`spell`) — the
skill loads this core at 0 tokens until invoked.
