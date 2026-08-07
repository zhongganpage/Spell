# Spell — An Adversarial Multi-Agent Prover

Spell is a working protocol for proving mathematical theorems with multiple
AI agents that **distrust each other**. Feed it a rough idea; it returns a
manuscript — in bounded runs (quick 30 min · medium 60 min · long 2 h · superlong 4 h), with you deciding between runs.

> **Input:** a rough idea → **Output:** a manuscript.

## What it is

A single agent works on the problem and writes up its results as a
**draft**. A **review panel** of adversarial agents attacks the draft, attacks
each other's attacks, and rebuts; a **ranking agent** weighs the whole record;
a separate **manuscript agent** writes the **manuscript** from that ranking.
An independent **high-level check** then judges novelty and sufficiency.
Only a manuscript that passes is delivered.

Every run is time-boxed to a **run envelope** you set at project start
(`quick` 30 min · `medium` 60 min · `long` 2 h · `superlong` 4 h; default
`medium`) and ends with a deliverable (draft,
manuscript, or progress report) plus a decision list for you — Spell never
runs autonomously for weeks; the dossier carries the state between runs, and
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
  stuck ladder with a minimum-output floor, and anti-give-up rules.
- **Review panel** — 5 agents, phases A–F: three reviewers (two internal, one
  **exterior agent** from a different provider via API or the local Codex
  CLI), then ranking, then manuscript.
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
2. Answer the startup questions: output form (PDF / LaTeX / Markdown / HTML),
   run envelope (`RUN_LENGTH`: `quick` 30 min · `medium` 60 min · `long` 2 h ·
   `superlong` 4 h; default `medium`), and the exterior agent (`X_PROVIDER`,
   `X_MODEL`, `X_ACCESS` — a provider API key or the local Codex CLI).
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
  per run — `quick` 30 min · `medium` 60 min · `long` 2 h · `superlong` 4 h
  (chosen at project start) — every run ends with a deliverable and a decision
  list for you.
- **Recording is mandatory.** Progress is measured in dossier entries, not
  solutions; dead ends and rejections are data.
- **Nothing is a certificate.** Panel verdicts and high-level checks are
  independent review, not formal proof.

## Provenance

The protocol synthesizes two research-protocol ideas — the *Persistence
Protocol* (never give up, record everything) and the *Independent
Verification Protocol* (no agent grades its own homework) — adapted to a
multi-agent pipeline. `reference/self-review.md` records a two-agent critical
review of the design and the decisions taken since.
