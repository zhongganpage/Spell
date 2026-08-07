# Spell — An Adversarial Multi-Agent Proof Protocol

Spell is a set of working rules for proving mathematical theorems with
multiple AI agents that distrust each other. You give it a rough idea; it
returns a finer manuscript.

> **Input:** a rough idea → **Output:** a manuscript.
>
> Each run is time-boxed to a run envelope (`RUN_LENGTH`) set at project
> start; longer projects span several runs with
> your decisions in between.

Spell is a proof protocol, not a proof checker: it does not run Lean/Coq
verification — the output is reviewed prose, never a machine-checked proof.
Although it does not completely prove the theorems for you, it helps you
materialize the rough ideas properly in the current literature and visualize
the potential gaps. You can use it for multiple rounds and add new ideas when
needed to deepen your thoughts.

## How it works

A single agent works on the problem and writes up its results as a **draft**.
A **review panel** of three adversarial agents — A1 (counterexample hunter),
A2 (step validator), and either an **exterior agent X** from another
company's agent API or, when there is no X, an internal **A3** (architecture
critic) — attacks the draft, attacks each other's attacks, and rebuts; a
**ranking agent** then weighs the whole record and ranks its ideas, and a
separate **manuscript agent** writes the **manuscript** from that ranking. An independent
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
| `definition.md` | The vocabulary: rough idea, draft, manuscript, progress report, exterior agent, review report, rebuttal, ranking, high-level check. Also fixes the **output form** and the **run envelope** (`RUN_LENGTH`), asked once at project start. |
| `review-panel.md` | The five-agent panel — 2 internal reviewers + 1 exterior reviewer (external agent API) + 1 ranking agent + 1 manuscript agent; phases A–F. |
| `high-level-check.md` | The independent novelty/sufficiency gate. |
| `protocol.md` | The working protocol: dossier, attempts log, transformation toolkit, stuck ladder, anti-give-up rules, delegation of tedious work (drafts stay high-level; manuscript sub-agents fix contradictions), verification ledger, startup checklist. |

## Starting a project

1. Read `definition.md` and `protocol.md` (especially §10, the startup
   checklist).
2. **Choose the exterior reviewer — the very first question.** Either
   configure X — `X_PROVIDER`, `X_MODEL`, `X_ACCESS` (provider API key, or
   the locally installed Codex CLI; keys live in environment variables,
   never in the dossier) — or declare no X, and the panel runs internal
   A1 + A2 + A3 with explicit roles.
3. Tell Spell the **output form** (PDF / LaTeX / Markdown / HTML) — it asks
   once, at the beginning.
4. Fix the **run envelope** (`RUN_LENGTH`, chosen at project start).
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
  per run (chosen at project start); every run ends with a deliverable and a
  decision list for you.
- **Recording is mandatory.** Progress is measured in dossier entries, not
  solutions; dead ends and rejections are data.
- **Portable to any agentic system.** Spell is roles, phases, and rules — not
  a model or a harness — so it layers onto any agentic system and is easy to
  personalize: swap prompts, add tools, change phases, or reuse the
  discipline elsewhere.
- **Nothing is a certificate.** Panel verdicts and high-level checks are
  independent review, not formal proof.
