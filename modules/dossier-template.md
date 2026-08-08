# Spell — Dossier Template

Copy this file per problem. Fill in the locked section once; the rest grows
by appending. Rules: append-only, dated, notation-locked, versioned, every
entry ends with a next step.

Artifacts live in `<project root>/research/`; this dossier and `question.md`
stay at the top. Once a kind — drafts, manuscripts, reports (incl. panel
records), or change lists — has **more than 2 versions**, move it into its
own folder (`research/drafts/`, `research/manuscripts/`,
`research/reports/`, `research/changelogs/`): all its versions move there,
new ones are written into the folder.

```markdown
# Dossier: <short problem name>

**Status:** OPEN — 0 threads active | **Last session:** <date>

## Problem statement (locked)

<precise statement, all definitions>

**Statement version:** Q v1 (locked <date>) — every update is a new version
(`Q v2`, …), dated, with a note on what changed and why. Never rewritten.

**Output form:** <PDF | LaTeX | Markdown | HTML — asked at project start>

**Run envelope:** RUN_LENGTH=<value chosen at project start>

**Rounds:** ROUNDS=<n> (decided <date>) — 1–10, recommended ≤ 10

**Exterior panel agent:** X_PROVIDER=<provider> · X_MODEL=<model> ·
X_ACCESS=<api | codex> (credentials in environment variables or a secrets
store, never in this dossier)

**Internal record format:** <LaTeX | Markdown>

**question.md:** <path — the canonical statement, converted from the input
document at project start; dated `Subgoals (round N)` sections appended after
each round>

## Notation

- <symbol> : <meaning>

## Examples & computations

| date | case | computed value / observation | pattern? |

## Reformulations & connections

- **R1.** <form of the problem> — useful because <reason> (<date>)

## Literature map

| date | source | theorem / claim | hypotheses | gap it leaves |

## Knowledge State (rewritten at session end — the only rewritten section)

**Conjectures registry**

| id | conjecture | status (active/supported/refuted/parked-until) | depends on | last touched |

**Obstructions register**

| id | obstruction | created by (thread, move) | status |

**Champion draft:** <artifact + version>

**Dependency backlinks:** claim → artifacts building on it (a `counterexample`
flags its dependents `affected`; they are re-verified before reuse).

## Attempts log

### <date> — thread <T#>, move <M#>
tried:   ...
broke:   ...
implies: ...
next:    ...

## Delegated tasks

| date | task (what was asked) | sub-agent (model/backend) | result / where recorded | implications |

## Claims & verification ledger

| date | claim | status | formalized (Y/N) | reviewer (role, model) | verdict & reasons | repair targets / notes |

## Panel & check ledger

| date | round | mode (normal/fast) | artifact (version) | changelog (version) | panel (normal: A1,A2,X/A3,R,M · fast: A+B) | panel verdict | high-level check | routing |

> In the panel cell, record which agents actually ran and the roles — e.g.
> `X:ok` or `X:unavailable — A1+A2+A3 roles, confidence downgraded`. Mark the
> mode (`normal` / `fast`) and the streamline run (`S:ok`); a fast round
> carries a confidence downgrade. Fast rounds record the high-level check
> as `skipped (fast mode)` — the check runs in normal mode only.

## Round timing

The orchestrator writes timestamps **live, not backfilled**, for every
round in every mode (normal and fast): `Round N started <ISO timestamp>` in
the dossier before any agent spawns, and a timestamp each time a phase
starts or ends. At round end it computes the elapsed time per phase and the
total from those timestamps, fills the table, reports them in the round's
deliverable + decision list, and **shows the round time to the user at the
end of the round** (closing message: total + phase breakdown). Fast rounds
have no check phase.

| round | started | ended | elapsed (total) | phase breakdown (loop / draft / panel / manuscript / streamline / check — check is normal-mode only) |

| 1 (fast) | 2026-08-08T08:55:00+08:00 | 2026-08-08T09:05:00+08:00 | 10 min | draft 1m · attack 3m · manuscript 3m · streamline 3m (no check) |
| 1 (normal) | 2026-08-08T09:10:00+08:00 | 2026-08-08T09:24:00+08:00 | 14 min | draft 1m · attack 3m · manuscript 3m · streamline 3m · check 4m |

## Open threads / next steps

- **T1** <thread description> — state: active — next: <concrete step>
- **T2** <thread description> — state: stalled <date> — resume: <one line>

## Delivery note (one page, per delivered manuscript)

Models per role and diversity achieved · checks run and verdicts · claims
still `claimed`/`under-review` · computed vs. opined · fetched vs. remembered
· formalization status · mode (normal/fast) and any confidence downgrade
```

Example attempts-log entry:

```markdown
### 2026-08-06 — thread T1, move M3 (specialize to n = 2)
tried:   proved the claim for the 2-variable case by explicit expansion.
broke:   the expansion needs a commutation property that fails for n >= 3.
implies: the obstruction is noncommutativity, not size — try commuting
         generators, or a nilpotent family, next.
next:    state and test the "commuting generators" subcase; search
         literature for "commutative case <problem name>".
```
