# Spell — Dossier Template

Copy this file per problem. Fill in the locked section once; the rest grows
by appending. Rules: append-only, dated, notation-locked, every entry ends
with a next step.

```markdown
# Dossier: <short problem name>

**Status:** OPEN — 0 threads active | **Last session:** <date>

## Problem statement (locked)

<precise statement, all definitions>

**Output form:** <PDF | LaTeX | Markdown | HTML — asked at project start>

**Run envelope:** RUN_LENGTH=<quick | medium | long | superlong> — 30 min · 60 min · 2 h · 4 h per run (chosen at project start)

**Exterior panel agent:** X_PROVIDER=<provider> · X_MODEL=<model> ·
X_ACCESS=<api | codex> (credentials in environment variables or a secrets
store, never in this dossier)

**Internal record format:** <LaTeX | Markdown>

## Notation

- <symbol> : <meaning>

## Examples & computations

| date | case | computed value / observation | pattern? |

## Reformulations & connections

- **R1.** <form of the problem> — useful because <reason> (<date>)

## Literature map

| date | source | theorem / claim | hypotheses | gap it leaves |

## Attempts log

### <date> — thread <T#>, move <M#>
tried:   ...
broke:   ...
implies: ...
next:    ...

## Claims & verification ledger

| date | claim | status | reviewer (model) | verdict & reasons | repair targets / notes |

## Panel & check ledger

| date | artifact | panel (A1,A2,X,R,M) | panel verdict | high-level check | routing |

> In the panel cell, record which agents actually ran — e.g. `X:ok` or
> `X:unavailable — reduced diversity`.

## Open threads / next steps

- **T1** <thread description> — state: active — next: <concrete step>
- **T2** <thread description> — state: stalled <date> — resume: <one line>
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
