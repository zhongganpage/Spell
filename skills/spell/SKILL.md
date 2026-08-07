---
name: spell
description: Run Spell, the adversarial multi-agent prover for mathematical theorem proofs — rough idea in, manuscript out, in runs of at most 8 hours with user decisions between runs. Use when the user asks to run Spell, prove or review a mathematical theorem, turn a rough idea into a manuscript, run the review panel or high-level check on a draft, or work on an open problem under Spell's protocol.
---

# Spell — Core

Adversarial multi-agent prover for mathematical theorem proofs. **Contract:**
rough idea in → manuscript out. **Run envelope:** ≤ 8 h of agent work per
run; every run ends with a deliverable + a decision list for the user; the
dossier carries state across runs; the user decides between runs. Never run
autonomously across days.

## Vocabulary
- **draft** — one agent's single-time report; never delivered.
- **manuscript** — post-panel report; the deliverable (output form chosen at
  project start: PDF / LaTeX / Markdown / HTML).
- **progress report** — run summary (tried / stands / ruled out) + decision list.
- **review report · cross-judgement · rebuttal · ranking** — panel records.

## Pipeline
working loop → draft → review panel → manuscript → high-level check →
deliverable + decisions. Full detail in the repository's `reference/` folder
(github.com/zhongganpage/Spell); paste-ready prompts in `./modules/prompts.md`.

## Working loop
LOAD dossier → ATTACK one toolkit move (`./modules/toolkit.md`) → RECORD
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
This package: `SKILL.md` (this file — always load) · `modules/` (on-demand,
siblings of this file). The full authoritative rules live in the repository's
`reference/` folder. Install by copying this folder to the skill directory of
your agent CLI (see the repository README, "Installing as a skill").
