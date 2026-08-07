# Spell — An Adversarial Multi-Agent Prover

Spell is a set of working rules for proving mathematical theorems with
multiple AI agents that distrust each other. You give it a rough idea; it
returns a manuscript.

> **Input:** a rough idea → **Output:** a manuscript.
>
> Each run is time-boxed to a run envelope set at project start (`RUN_LENGTH`: `quick` 30 min · `medium` 60 min · `long` 2 h · `superlong` 4 h; default `medium`); longer projects span several runs with
> your decisions in between.

## How it works

A single agent works on the problem and writes up its results as a **draft**.
A **review panel** of three adversarial agents — two internal and one
**exterior agent** called through another company's agent API — attacks the
draft, attacks each other's attacks, and rebuts; a **ranking agent** then
weighs the whole record and ranks its ideas, and a separate **manuscript
agent** writes the **manuscript** from that ranking. An independent
**high-level check** then judges whether the manuscript is novel (against the
literature) and sufficient (the gaps are reachable). Only a manuscript that
passes is delivered.

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

Every run ends with a **deliverable** (draft, manuscript, or progress
report) and a **decision list** for you — Spell never runs autonomously for
weeks; you decide between runs, and the dossier carries the state across
them.

The working loop runs on the **protocol** (`protocol.md`), which synthesizes
two source protocols — the *Persistence Protocol* (never give up, record
everything) and the *Independent Verification Protocol* (no agent grades its
own homework).

## Files

| File | Role |
|---|---|
| `README.md` | This overview. |
| `definition.md` | The vocabulary: rough idea, draft, manuscript, progress report, exterior agent, review report, rebuttal, ranking, high-level check. Also fixes the **output form** and the **run envelope** (`RUN_LENGTH` — `quick` 30 min · `medium` 60 min · `long` 2 h · `superlong` 4 h), asked once at project start. |
| `review-panel.md` | The five-agent panel — 2 internal reviewers + 1 exterior reviewer (external agent API) + 1 ranking agent + 1 manuscript agent; phases A–F. |
| `high-level-check.md` | The independent novelty/sufficiency gate. |
| `protocol.md` | The working protocol: dossier, attempts log, transformation toolkit, stuck ladder, anti-give-up rules, verification ledger, startup checklist. |

## Starting a project

1. Read `definition.md` and `protocol.md` (especially §10, the startup
   checklist).
2. Tell Spell the **output form** (PDF / LaTeX / Markdown / HTML) — it asks
   once, at the beginning.
3. Fix the **run envelope** (`RUN_LENGTH`: `quick` 30 min · `medium` 60 min ·
   `long` 2 h · `superlong` 4 h; default `medium`).
4. Configure the **exterior panel agent** — set the variables `X_PROVIDER`,
   `X_MODEL`, `X_ACCESS` (provider API key, or the locally installed Codex
   CLI; keys live in environment variables, never in the dossier).
5. Create the dossier: lock the problem statement, fix the notation, open
   the first thread.
6. Input the rough idea. From then on: draft → panel → high-level check →
   deliverable + decision list, with the working loop in between whenever a
   check fails.

## Design principles

- **Adversariality is a feature.** Every claim is attacked; every attack is
  itself attacked and rebutted; the ranking and manuscript agents must
  resolve the whole record.
- **Independence is structural.** Reviewers never see the author's reasoning,
  each runs in a fresh context, and one reviewer (X) comes from a different
  company's model entirely.
- **Runs are bounded, the user decides.** At most `RUN_LENGTH` of agent work
  per run — `quick` 30 min · `medium` 60 min · `long` 2 h · `superlong` 4 h
  (chosen at project start); every run ends with a deliverable and a decision
  list for you.
- **Recording is mandatory.** Progress is measured in dossier entries, not
  solutions; dead ends and rejections are data.
- **Nothing is a certificate.** Panel verdicts and high-level checks are
  independent review, not formal proof.
