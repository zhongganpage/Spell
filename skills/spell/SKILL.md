---
name: spell
description: Spell is a universal proof enhancement protocol based on adversarial multi-agent review procedures. You can use it on ANY agentic systems such as Claude Code, CodeX, Kimi Code and etc. My personal experience is that for Kimi Code + Deepseek-v4-flash, it will output a fine manuscript within 2 hours and the cost is less than 1 US dollar. Have fun :)
---

# Spell — Core

Adversarial multi-agent proof protocol for mathematical theorems. **Contract:**
rough idea in → finer manuscript out. No Lean/Coq verification — the output
is reviewed prose, never a machine-checked proof.
**Run envelope:** a variable `RUN_LENGTH`
chosen at project start, bounding the agent work per run; every run ends
with a deliverable + a decision list for the user; the dossier carries state
across runs; the user decides between runs. Never run autonomously across
days.

## Vocabulary
- **draft** — one agent's single-time report, versioned (`v1`, `v2`, …);
  high-level by design — tedious work is delegated to sub-agents, only its
  outcomes appear; never delivered.
- **manuscript** — post-panel report; the deliverable (output form chosen at
  project start: PDF / LaTeX / Markdown / HTML); versioned, names the draft
  version it improves on.
- **progress report** — run summary (tried / stands / ruled out) + decision
  list; versioned.
- **review report · cross-judgement · rebuttal · ranking** — panel records.

## Pipeline
working loop → draft → review panel → manuscript → high-level check →
deliverable + decisions. Full detail in the repository's `reference/` folder
(github.com/zhongganpage/Spell); paste-ready prompts in `./modules/prompts.md`.

## Project start
The **very first question** is a choice: configure the exterior reviewer X
(`X_PROVIDER` / `X_MODEL` / `X_ACCESS`, `./modules/providers.md` — provider
API env var or the local Codex CLI; default kimi/k2.7/api), or declare **no
X** — the panel then runs internal A1 + A2 + A3 with explicit roles. Then the
output form (PDF / LaTeX / Markdown / HTML), then the dossier. Ask these
before any work starts; a no-X panel records `X unavailable — reduced
diversity` and a confidence downgrade.

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
5. **Everything is versioned.** Every draft, report, manuscript, and update
   to the problem statement carries a version (`v1`, `v2`, …); cite the
   version you build on.
6. **Delegate the tedious; stay high-level.** A massive or tedious task that
   still leaves distance to the goal → spawn a sub-agent, keep thinking
   high-level, record its result in the dossier. Draft sub-agents record
   contradictions honestly; manuscript sub-agents fix contradictions between
   the panel agents.
7. **Measure the panel.** Periodic auditor reads the ledgers (ritualism, gate
   leakage, per-reviewer agreement); canary panels seed known-false claims to
   measure detection. Record what you find.

## Review panel — 5 agents, phases A–F
A1 counterexample-hunter + A2 step-validator (internal) + **X** exterior
independent (variables `X_PROVIDER` / `X_MODEL` / `X_ACCESS`; access =
provider API env var or the local Codex CLI) — or internal **A3**
architecture-critic when there is no X + R ranking + M manuscript.

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
