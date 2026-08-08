# Spell — The High-Level Check

An independent gate between "we have an argument" and "we deliver it". It
answers two high-level questions about the input draft or manuscript:

1. **Novel** — is this new compared to the literature?
2. **Sufficient** — is the approach feasible, or does it rest on gaps far
   beyond reach?

It deliberately does **not** check the line-by-line correctness of the proof;
that is the review panel's adversarial job and the verification ledger's job.

Its verdict is **descriptive routing** — to the next verification depth
(`screening → panel → formalization`) or to the user — never a certificate
(`depth escalation` in `definition.md`). On negative rounds the check is
replaced by the **negative-value assessment**
(§ "Negative rounds — the negative-value assessment").

## When it runs

- **Required** on every manuscript in **normal mode**, before delivery —
  after the manuscript pass (**streamline folded into M**; `review-panel.md`):
  the check receives the surgical manuscript and its change list (the change
  list records what M changed). **Fast mode skips the check** — a fast round
  delivers after the manuscript pass, with the fast-mode confidence
  downgrade.
- **Replaced on negative rounds.** A negative round does not run the
  high-level check at all; its outcome is delivered as a **negative-value
  assessment** (§ "Negative rounds — the negative-value assessment"),
  produced by a dedicated cheap agent that runs in **all tiers** — fast and
  screening rounds do not skip it. It replaces the check for negative
  outcomes; it is not a form of it.
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
and version it judged (e.g. `manuscript v1`). The check agent must write the
report to the assigned file and confirm the write in its final message; a
read-only check agent (or a write-blocked sandbox) delivers the report text
in its final message and the orchestrator persists it verbatim
(`protocol.md` §9). A `fail` report must also name the **revival triggers**
it attaches — the `re-examine when <event>` conditions (§ "Decision
routing").

### Novelty

Every verdict rests on **fetched sources, not recall**: the literature sweep
is actually executed (an M11/M12 sub-agent fetches and reads), and every
literature claim carries a locator (arXiv ID / DOI / page). A `novel` or
`known` verdict without a fetched, locator-carrying source is unsupported —
at most `conditional`.

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

- `pass` — novel and within reach: the document routes forward (delivery to
  the user; its surviving claims to the next verification depth).
- `conditional` — viable idea, but novelty or reach is unclear or shaky:
  list exactly what must be established first.
- `fail` — known, or beyond reach, or the high-level architecture does not
  support the claim: say why in three sentences or fewer, and name the
  revival triggers (the `re-examine when <event>` conditions).

A verdict is **descriptive routing**, never a certificate: it routes to the
next verification depth (`screening → panel → formalization`) or to the
user — a `pass` routes forward, it does not certify the mathematics.

## Decision routing

| Overall | Route |
|---|---|
| `pass` | Routes forward: the manuscript is delivered to the user, and its surviving claims move to the next verification depth (`screening → panel → formalization`). |
| `conditional` | A **user decision point**, never an automatic continue: the user chooses **repair** (scope frozen — fix the listed conditions only, no new conjectures; re-check only those; the artifact is not re-derived), **park**, or **re-scope**. |
| `fail` | **Demotes** the direction (portfolio rank down) and attaches **revival triggers** ("re-examine when <event>", wired to the literature watch list and new dossier evidence); the report names them. |

No verdict auto-parks a thread: the old automatic "reformulate or park"
routing is removed. **Parking is an explicit user decision** — and even then
the fragments rule applies. The panel's `misdirected` verdict demotes and
attaches revival triggers the same way (`review-panel.md`).

**The only terminal state for an idea is a demonstrated counterexample with
a reproducible computation.** Under the **fragments rule**, every terminal
verdict — a counterexample or an evidence-backed rejection — deposits the
maximal true subcase, the obstruction, and the closest technique into the
wild-ideas register, with revival triggers.

A `fail`ed or `conditional` manuscript is **not** delivered and **not**
silently kept as a manuscript: whatever the user decides next — repair,
park, re-scope, or re-entry into the working loop — the disposition and its
reason are recorded in the verification ledger as a dated event.

## Negative rounds — the negative-value assessment

A negative round's deliverable is a **negative-value assessment**: did we
truly rule out the direction, or did the tool just fail? It is produced by a
**dedicated cheap agent** that runs in **all tiers** — fast and screening
rounds do not skip it. For negative outcomes it **replaces** the high-level
check; it is not a form of it.

Its job is to **assess — never certify** — the evidence quality of the
rule-out, weighing exactly what stands behind it:

- **exact cases** — reproduced exact-case computations;
- **fetched literature** — fetched, locator-carrying sources;
- **verified computation** — reproducible scripts, with role separation for
  load-bearing numbers (the numerics guardrail).

Verdicts:

- `ruled-out-with-evidence` — the direction is dead, and the evidence
  (exact cases, fetched literature, verified computation) stands.
- `tool-failure` — the rule-out is not established: the tool failed, so the
  direction remains open.

A rule-out with assessed evidence is a **first-class deliverable** (the
negative round delivers it instead of a manuscript) and is recorded in the
**wild-ideas register** under the fragments rule: the maximal true subcase,
the obstruction, and the closest technique, each with revival triggers.

## Interaction with verification

The high-level check is a strategic gate, not a correctness certificate. Even
a `pass` manuscript contains claims that remain `claimed`/`under-review`
until the independent verification session accepts them. The check never
overrides a rejected claim, and a `fail` never depends on a single step —
only on the idea's novelty and reach.

Verification depth is per-claim and non-decreasing across rounds
(`screening → panel → formalization`): a claim that survives round N is not
re-checked at the same depth in round N+1. The gate's verdict routes to the
next depth or to the user — it never certifies.
