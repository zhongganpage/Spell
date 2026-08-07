# Spell — An Adversarial Multi-Agent Proof Protocol

Spell is a **universal proof enhancement protocol** based on adversarial
multi-agent review procedures. You can use it on ANY agentic systems such as
Claude Code, CodeX, Kimi Code and etc. My personal experience is that for
Kimi Code + Deepseek-v4-flash, it will output a fine manuscript within 2
hours and the cost is less than 1 US dollar. Have fun :)

> **Input:** a rough idea → **Output:** a manuscript.

Spell is a **proof protocol**, not a proof checker: it does not run Lean/Coq
verification — the output is reviewed prose, never a machine-checked proof.
Although it does not completely prove the theorems for you, it helps you
materialize the rough ideas properly in the current literature and visualize
the potential gaps. You can use it for multiple rounds and add new ideas when
needed to deepen your thoughts.

## What it is

A single agent works on the problem and writes up its results as a
**draft**. A **review panel** of adversarial agents attacks the draft, attacks
each other's attacks, and rebuts; a **ranking agent** weighs the whole record;
a separate **manuscript agent** writes the **manuscript** from that ranking.
An independent **high-level check** then judges novelty and sufficiency.
Only a manuscript that passes is delivered.

Every run ends with a deliverable (draft,
manuscript, or progress report) plus a decision list for you; the dossier carries the state between runs, and
you decide between them.

## How it works

```
rough idea
   │
   ▼
working loop ────► draft ──► review panel ──► manuscript ──► high-level check
   ▲                    │                        │               │  │
   └────────────────────┴────────────────────────┴───────────────┘  │
                     (fail / conditional → back to the loop)         ▼
                                                          deliverable + decisions
                                                          → you decide → next run
```

- **Working loop** — persistence protocol: dossier, attempts log, a Pólya-style
  transformation toolkit (compute examples, specialize, reformulate, …), a
  stuck ladder with a minimum-output floor, anti-give-up rules, and delegation
  of tedious work to sub-agents (drafts stay high-level; manuscript sub-agents
  fix contradictions between agents).
- **Review panel** — 5 agents, phases A–F: three reviewers (A1
  counterexample-hunter, A2 step-validator, and X — an **exterior agent**
  from a different provider, or an internal A3 architecture-critic when you
  have no X), then ranking, then manuscript.
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
├── core.md                distilled rules — the only thing loaded per run (~0.8k tokens)
├── skills/spell/          the skill package — copy this folder to install
├── modules/               on-demand details
│   ├── toolkit.md             transformation toolkit M1–M11 + stuck ladder
│   ├── prompts.md             5 paste-ready prompts (reviewer, panel, ranking, manuscript, check)
│   ├── dossier-template.md
│   └── providers.md           exterior agent variables + provider table
└── reference/             authoritative full rules (archive; read only to audit)
    ├── README.md · definition.md · review-panel.md
    ├── high-level-check.md · protocol.md · self-review.md
```

## Installing as a skill

Spell ships as a **self-contained skill package** in `skills/spell/`:
`SKILL.md` (the distilled core, ~0.8k tokens) plus the modules it references.
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
   Markdown / HTML).
3. Create the dossier from `modules/dossier-template.md`: lock the problem
   statement and notation, open the first thread.
4. Input the rough idea. From then on: draft → panel → high-level check →
   deliverable + decision list.

## Design principles

- **Adversariality is a feature.** Every claim is attacked; every attack is
  itself attacked and rebutted.
- **Independence is structural.** Reviewers never see the author's reasoning,
  each runs in a fresh context, and one reviewer comes from a different
  company's model entirely.
- **Runs are bounded; the user decides.** At most `RUN_LENGTH` of agent work
  per run (chosen at project start) — every run ends with a deliverable and a
  decision list for you.
- **Recording is mandatory.** Progress is measured in dossier entries, not
  solutions; dead ends and rejections are data.
- **Everything is versioned.** Every draft, report, and manuscript carries a
  version (`v1`, `v2`, …), and so does every update to the problem statement;
  nothing is cited without its version.
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
