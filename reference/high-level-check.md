# Spell — The High-Level Check

An independent gate between "we have an argument" and "we deliver it". It
answers two high-level questions about the input draft or manuscript:

1. **Novel** — is this new compared to the literature?
2. **Sufficient** — is the approach feasible, or does it rest on gaps far
   beyond reach?

It deliberately does **not** check the line-by-line correctness of the proof;
that is the review panel's adversarial job and the verification ledger's job.

## When it runs

- **Required** on every manuscript, before delivery.
- **Optional** on an early draft, as a triage check — catching "this is
  already known" or "this cannot possibly work" before expensive panel time
  is spent.

## Independence

The high-level check agent is a single new agent, run under the same
independence rules as the panel:

1. Fresh context; no knowledge of how the document was derived.
2. Receives only the document, the locked problem statement, and the
   definitions it uses. Not the author's confidence, not the panel's drama.
3. A different agent/backend/model when the harness allows it.
4. No expected outcome is communicated.

## What it checks

The check inspects only the **high-level ideas and how well they are
implemented** — not every detail:

1. **Novelty.** The central idea, stated plainly, is searched against the
   literature: nearest known results, related areas, whether the result (or
   its negation) is already known. It must distinguish "novel" from "new
   packaging of a known theorem".
2. **Sufficiency.** The crucial gaps: which steps are proved, which are
   labelled gaps, and whether the unproved core is (a) within reach of known
   tools, (b) plausible but hard, or (c) far beyond current reach. It also
   judges whether the shape of the idea — main theorem vs. the tools used —
   is commensurate with the goal.
3. **Implementation quality at the high level.** Is the architecture of the
   argument sound (would the claimed result actually follow if the gaps were
   filled)? Are the gaps honest (labelled `GAP:` and not smuggled in)?

## The report

The report is a single page of verdicts and reasons; it names the artifact
and version it judged (e.g. `manuscript v1`).

### Novelty

- `novel` — the central idea is not in the literature searched.
- `extension` — new, but visibly extends a known result; say which.
- `known` — the result, or an equivalent form, is already in the literature;
  cite it.

### Sufficiency

- `within reach` — the gaps are attackable with known tools.
- `plausible` — the gaps are hard; the approach might work but has no clear
  route yet.
- `beyond reach` — the approach rests on gaps far beyond current tools.

### Overall

- `pass` — novel and within reach: the document may be delivered.
- `conditional` — viable idea, but novelty or reach is unclear or shaky:
  list exactly what must be established first.
- `fail` — known, or beyond reach, or the high-level architecture does not
  support the claim: say why in three sentences or fewer.

## Decision routing

| Overall | Route |
|---|---|
| `pass` | Deliver the manuscript. |
| `conditional` | Return to the working loop with the listed targets; the revised artifact is a new **draft** (the panel runs again). |
| `fail` | Return to the working loop with the reason: the idea is reformulated (novelty/architecture failure) or parked in the dossier (reach failure). |

A failed or conditional manuscript is **not** delivered and **not** silently
kept as a manuscript: the downgrade to draft, and the reason, are recorded in
the verification ledger as a dated event.

## Interaction with verification

The high-level check is a strategic gate, not a correctness certificate. Even
a `pass` manuscript contains claims that remain `claimed`/`under-review`
until the independent verification session accepts them. The check never
overrides a rejected claim, and a `fail` never depends on a single step —
only on the idea's novelty and reach.
