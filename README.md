# Spell — An Adversarial Multi-Agent Prover

Spell is a working protocol for proving mathematical theorems with multiple
AI agents that **distrust each other**. Feed it a rough idea; it returns a
manuscript — in runs of at most 8 hours, with you deciding between runs.

> **Input:** a rough idea → **Output:** a manuscript.

## What it is

A single agent works on the problem and writes up its results as a
**draft**. A **review panel** of adversarial agents attacks the draft, attacks
each other's attacks, and rebuts; a **ranking agent** weighs the whole record;
a separate **manuscript agent** writes the **manuscript** from that ranking.
An independent **high-level check** then judges novelty and sufficiency.
Only a manuscript that passes is delivered.

Every run is time-boxed to **8 hours** and ends with a deliverable (draft,
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
├── core.md                distilled rules — the only thing loaded per run (~0.8k tokens)
├── modules/               on-demand details
│   ├── toolkit.md             transformation toolkit M1–M11 + stuck ladder
│   ├── prompts.md             5 paste-ready prompts (reviewer, panel, ranking, manuscript, check)
│   ├── dossier-template.md
│   └── providers.md           exterior agent variables + provider table
└── reference/             authoritative full rules (archive; read only to audit)
    ├── README.md · definition.md · review-panel.md
    ├── high-level-check.md · protocol.md · self-review.md
```

Registered as a user-scope **Skill** (`spell`) for agent CLIs that support
skills: 0 tokens until invoked.

## Starting a project

1. Load `core.md` (or invoke the `spell` skill).
2. Answer the startup questions: output form (PDF / LaTeX / Markdown / HTML),
   run envelope (default 8 h), and the exterior agent (`X_PROVIDER`,
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
- **Runs are bounded; the user decides.** At most 8 hours of agent work per
  run; every run ends with a deliverable and a decision list for you.
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
