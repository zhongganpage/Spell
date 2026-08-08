# Spell — An Adversarial Multi-Agent Proof Protocol

Spell is a **universal proof enhancement protocol** based on adversarial
multi-agent review procedures. You can use it on ANY agentic systems such as
Claude Code, CodeX, Kimi Code and etc. Every round runs in **normal mode**
(5-agent panel, phases A–F, plus the high-level check) or **fast mode** (a
2-agent A/B attack/rebut loop that skips the high-level check). My personal experience is that for
Kimi Code + Deepseek-v4-flash, it will output a fine manuscript within 2
hours and the cost is less than 1 US dollar. Have fun :)

> **Input:** a rough idea or a manuscript → **Output:** a finer manuscript.

Spell is a **proof protocol**, not a proof checker: it does not run Lean/Coq
verification — the output is reviewed prose, never a machine-checked proof.
Although it does not completely prove the theorems for you, it helps you
materialize the rough ideas properly in the current literature and visualize
the potential gaps. You can use it for multiple rounds and add new ideas when
needed to deepen your thoughts. You can also personalize it easily by using
agents.

## What it is

A single agent works on the problem and writes up its results as a
**draft**. A **review panel** of adversarial agents attacks the draft, attacks
each other's attacks, and rebuts; a **ranking agent** weighs the whole record;
a separate **manuscript agent** writes the **manuscript** from that ranking.
A **streamline agent** then tightens the manuscript before an independent
**high-level check** judges novelty and sufficiency.
Only a manuscript that passes is delivered.

Every run ends with a deliverable (draft,
manuscript, or progress report) plus a decision list for you; the dossier carries the state between runs, and
you decide between them.

## How it works

**First round, normal mode** — the 5-agent panel (A1, A2, X/A3, R, M;
phases A–F) reviews the draft, and the high-level check gates delivery
(`pass` → deliver; `conditional` / `fail` → back to the working loop).

```
rough idea ────────────────► working loop ────► draft ──┐
                                                        │
                                                        ▼
manuscript (input) ─────────────────────────────────► review panel ──► manuscript (+ change list)
                                                                        │
                                                                        ▼
                                                                 streamline (agent S)
                                                                        │
                                                                        ▼
                                                           high-level check (novelty + sufficiency)
                                                                        │
                                                                        ▼
                                                               deliverable + decisions
                                                                        │
                                                                        ▼
                                                                 you decide → next run
```

**Inside the review panel (normal mode)** — phases A–F, five agents; all run
in the background, and each phase starts only when its written inputs exist.

```
draft / input manuscript
  │
  ▼
Phase A — independent review (three reviewers, in parallel)
  ├── A1 counterexample-hunter ──► review report
  ├── A2 step-validator ─────────► review report
  └── X (exterior) or A3 (architecture critic) ──► review report
  │
  ▼
Phase B — exchange: each reviewer reads the other two reports
  │
  ▼
Phase C — cross-review: each judges the other two ──► cross-judgements
  │
  ▼
Phase D — rebuttal: each answers the criticisms of its own report
  │
  ▼
Phase E — R (ranking agent): reads the full record, ranks the ideas,
          closes the three reviewers
  │
  ▼
Phase F — M (manuscript agent): writes the manuscript + change list
  │
  ▼
streamline (S) ──► high-level check ──► deliverable + decisions
```

- **Working loop** — persistence protocol: dossier, attempts log, a Pólya-style
  transformation toolkit (compute examples, specialize, reformulate, …), a
  stuck ladder with a minimum-output floor, anti-give-up rules, and delegation
  of tedious work to sub-agents (drafts stay high-level; manuscript sub-agents
  fix contradictions between agents). A **manuscript input** skips this loop
  and the draft, entering the review panel directly — shorter rounds. The
  input document is converted into `question.md` at round 1, and each round
  appends its subgoals to it.
- **Review panel** — 5 agents, phases A–F: three reviewers (A1
  counterexample-hunter, A2 step-validator, and X — an **exterior agent**
  from a different provider, or an internal A3 architecture-critic when you
  have no X), then ranking, then manuscript. All agents run in the background;
  every manuscript version comes with a **change list** (what changed, why,
  and which changes are important). A **streamline agent** then simplifies
  the manuscript before the **high-level check** (normal mode only); a
  per-round **fast mode** (asked at the start of each round) replaces the
  panel with a 2-agent attack/rebut loop and delivers after streamlining —
  no high-level check.
- **Verification** — every claim runs `claimed → under-review →
  accepted | rejected | counterexample`; no agent grades its own homework.
- **High-level check** — an independent gate on novelty and sufficiency
  before delivery. It is independent review, never a certificate.

## Repository layout

```
Spell/
├── README.md              this file
├── LICENSE                MIT
├── .gitignore             credentials stay out of git
├── core.md                distilled rules — the only thing loaded per run
├── skills/spell/          the skill package — copy this folder to install
├── modules/               on-demand details
│   ├── toolkit.md             transformation toolkit M1–M12 + stuck ladder
│   ├── prompts.md             paste-ready prompts (reviewer · panel · ranking · manuscript · check · streamline · fast mode)
│   ├── dossier-template.md
│   └── providers.md           exterior agent variables + provider table
└── reference/             authoritative full rules (archive; read only to audit)
    ├── README.md · definition.md · review-panel.md
    ├── high-level-check.md · protocol.md · self-review.md
```

## Installing as a skill

Spell ships as a **self-contained skill package** in `skills/spell/`:
`SKILL.md` (the distilled core) plus the modules it references.
An installed skill costs **0 tokens until invoked**; invoking loads only the
core. All references inside the package are relative, so it is portable.

**User scope — all projects on this machine:**

```bash
mkdir -p ~/.kimi-code/skills
cp -r skills/spell ~/.kimi-code/skills/
```

Then invoke it in any session — e.g. ask your agent to "run Spell".

**Project scope — one repository** (the layout Kimi Code discovers under
`<project>/skills/`):

```bash
cp -r skills/spell <your-project>/skills/spell
```

**Verify:** start an agent session and ask for Spell's core; you should see
the distilled rules. If your CLI does not discover the folder, load
`core.md` manually — it is the same content.

**Package layout:**

```
skills/spell/
├── SKILL.md          frontmatter + distilled core (the only thing loaded)
└── modules/          on-demand: toolkit · prompts · dossier-template · providers
```

## Starting a project

1. Load `core.md` (or invoke the `spell` skill).
2. The **very first startup question** is a choice: set up the exterior
   reviewer X — `X_PROVIDER`, `X_MODEL`, `X_ACCESS` (a provider API key or
   the local Codex CLI) — or declare no X (the panel then runs internal A1 +
   A2 + A3 with explicit roles). Then the output form (PDF / LaTeX /
   Markdown / HTML), then the round count (`ROUNDS`, 1–10 — recommended
   ≤ 10: the accumulated record can exceed the context window beyond that).
3. Create the dossier from `modules/dossier-template.md`: lock the problem
   statement and notation, open the first thread. Convert the input document
   (any format — `.tex`, `.md`, `.pdf`, …) into `question.md`, the canonical
   statement.
4. Input the rough idea or the manuscript. A manuscript input goes straight
   to the panel; from then on: draft → panel → manuscript → streamline →
   high-level check → deliverable + decision list (normal mode; fast
   rounds skip the high-level check).
5. Every round starts by asking the **round mode**: **normal mode** (the
   5-agent panel, phases A–F) or **fast mode** (a 2-agent A/B attack/rebut
   loop — the panel is not run in a fast round). The choice is recorded in
   the dossier and binds the whole round.

## Design principles

- **Adversariality is a feature.** Every claim is attacked; every attack is
  itself attacked and rebutted.
- **Independence is structural.** Reviewers never see the author's reasoning,
  each runs in a fresh context, and one reviewer comes from a different
  company's model entirely.
- **Runs are bounded; the user decides.** At most `RUN_LENGTH` of agent work
  per run (chosen at project start) — every run ends with a deliverable and a
  decision list for you. At round 1 you choose how many rounds to run
  (`ROUNDS`, 1–10; recommended ≤ 10 to stay inside the context window); each
  round records its start time and reports its elapsed time at the end.
- **Recording is mandatory.** Progress is measured in dossier entries, not
  solutions; dead ends and rejections are data.
- **Everything is versioned.** Every draft, report, manuscript, change list,
  and ranking carries a version (`v1`, `v2`, …), and so does every update to
  the problem statement; nothing is cited without its version. Once an
  artifact kind has more than 2 versions, it moves into its own folder
  (`research/drafts/` · `research/manuscripts/` · `research/reports/` ·
  `research/changelogs/`).
- **Portable to any agentic system.** Spell is roles, phases, and rules — not
  a model or a harness — so it layers onto any agentic system and is easy to
  personalize: swap prompts, add tools, change phases, or reuse the
  discipline elsewhere.
- **Nothing is a certificate.** Panel verdicts and high-level checks are
  independent review, not formal proof.

## Provenance

The protocol synthesizes two research-protocol ideas — the *Persistence
Protocol* (never give up, record everything) and the *Independent
Verification Protocol* (no agent grades its own homework) — adapted to a
multi-agent pipeline. `reference/self-review.md` records a two-agent critical
review of the design and the decisions taken since.
