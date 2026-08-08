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

Once this dossier exceeds **~30 KB** it splits by section —
`claims-ledger.md` (claims & verification ledger), `attempts-log.md`
(attempts log), `version-inventory.md` (artifact version records) — and the
top-level dossier stays as the navigation index: a fresh session reads
Knowledge State first, then follows its pointers into the split files.

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
each round. Stays **minimal** — the statement and the dated subgoal list
only; analysis lives in the dossier>

## Knowledge State (the navigation index — first thing a fresh session reads;
rewritten at session end — the only rewritten section)

**Conjectures registry**

| id | conjecture | status (active/supported/refuted/parked-until) | depends on | last touched |

**Obstructions register**

| id | obstruction | created by (thread, move) | status |

**Champion pointer:** <thread T# (GAP-owner) — artifact + version>

**Dependency backlinks:** claim → artifacts building on it (a `counterexample`
flags its dependents `affected`; they are re-verified before reuse).

When the dossier splits by section (> ~30 KB, see header), this section stays
in the top-level dossier as the index; the ledgers move to
`claims-ledger.md` / `attempts-log.md` / `version-inventory.md`.

## Notation

- <symbol> : <meaning>

## Examples & computations

| date | case | computed value / observation | pattern? |

## Reformulations & connections

- **R1.** <form of the problem> — useful because <reason> (<date>)

## Literature map

| date | source | theorem / claim | hypotheses | gap it leaves |

## Wild ideas register (speculative lane)

`speculative` ideas — conjectures, reformulations, analogies, technique
transfers — that are not yet claims. Exempt from the verification ledger and
from panel attack; they leave the register **only by nomination** (to
`promising`). **Bounded: ≤ 15 active**; the rest are archived with their
revival triggers and re-checked only when a trigger fires or the idea-yield
ranking promotes them. Fragments of every terminal verdict (fragments rule:
maximal true subcase, obstruction, closest technique) and every idea-sprint
candidate (survivors and discards) land here with revival triggers. Sprint
survivors also form the round's **sprint backlog** — the top of the attack
queue (Open threads below); a candidate settled by its discriminating test
is marked `settled (round N)` and leaves the backlog, the rest keep their
triggers.

| id | idea | source (thread/move) | status (active/archived) | revival trigger | last touched |

## Promising claims table

Nominated for development; may be built upon heuristically in exploration —
every use explicitly labeled `heuristic use of <claim vN>`. Never a premise
in any delivered artifact, never in user-facing reporting of established
results, and not subject to review until nominated `claimed`.

| id | claim | labeled heuristic uses | promoter/date | status |

## Numerics register

Every numerical claim carries an **exact-case reproduction** embedded in its
script; **load-bearing numbers carry two independent scripts**; role
separation — the agent that builds the theory is not the agent that writes
the verification scripts. All agents **cite the shared, versioned
norms/solver file**: the notation lock is enforced, not just claimed. The
scripts, their author roles, and their run logs are a **delivery
prerequisite**: the post-delivery check audits them (exist, agree,
role-separated) and the audit rows land in Post-delivery checks below.

| claim | exact-case reproduction (embedded) | independent script count | script paths | author role (≠ theory agent) | run log | verdict |

## Post-delivery checks

The background grounding audit run after delivery during the user's decision
point (off the round's critical path; `prompts.md` §16). Record the
novelty spot-checks (one in three of delivered `novel`/`extension` verdicts)
and the numerics audit per round.

| date | round | novelty spot-checked claims (verdict: novel-confirmed / known / extension-of, locator) | false-known / false-novel events | numerics audit (pass/fail per claim) | run log refs |

## Attempts log

### <date> — thread <T#>, move <M#>
tried:   ...
broke:   ...
implies: ...
next:    ...

## Delegated tasks

Writers (working-loop author, manuscript agent M) record their **plan** here
at the start of every session: the task list, which sub-agent owns each, and
the outcome. The draft/manuscript cites this table and marks every
integrated outcome; a writer that did not plan, distribute, and integrate
is in protocol violation.

| date | task (what was asked) | sub-agent (model/backend) | result / where recorded | implications |

## Claims & verification ledger

| date | claim | status | screening | depth | formalized (Y/N) | reviewer (role, model) | verdict & reasons | repair targets / notes |

> Every claim **nominated for the verification pipeline** (entering
> `claimed`/`under-review`) runs the claim-reviewer screening (`prompts.md`
> §1) before any panel decision; a screening fail returns to the loop as a
> repair task (the ≤ 2-repair rule applies). `speculative` and `promising`
> are exempt — they are not yet claims in the pipeline. `depth` is per-claim
> and non-decreasing across rounds (`screening → panel → formalization`): a
> claim that survives round N is not re-checked at the same depth in round
> N+1.

## Panel & check ledger

| date | round | tier (screening/fast/normal) | artifact (version) | changelog (version) | panel (normal: A1,A2,X/A3,R,M + promoter · fast: A+B · screening: claim-reviewer) | panel verdict | high-level check | routing |

> In the panel cell, record which agents actually ran and the roles — e.g.
> `X:ok` or `X:unavailable — A1+A2+A3 roles, confidence downgraded`. Mark the
> tier (`screening` / `fast` / `normal`); a fast round carries a confidence
> downgrade, a screening round produces no manuscript. Fast and screening
> rounds record the high-level check as `skipped (fast tier)` — the check
> runs in the normal tier only; negative rounds record the negative-value
> assessment in its place. The streamline step is folded into M (no
> standalone S).

## Idea-yield table

Computed from the ledgers each round; re-ranks the portfolio below.

| thread | accepted claims | promoted ideas | new subgoals | cost | yield |

## Portfolio table

Open threads as a ranked population with a fixed budget split — **70%
champion / 20% runners-up / 10% wild**. Re-ranked each round by the
idea-yield table; GAP-owner threads hold the champion share until the
ladder floor; the champion share changes only on a floor verdict or yield
ranking.

| thread | share (champion / runners-up / wild) | budget % |

## Round timing

The orchestrator writes timestamps **live, not backfilled**, for every
round in every tier (screening, fast, normal): `Round N started <ISO
timestamp>` in the dossier before any agent spawns, and a timestamp each
time a phase starts or ends. At round end it computes the elapsed time per
phase and the total from those timestamps, fills the table, reports them in
the round's deliverable + decision list, and **shows the round time to the
user at the end of the round** (closing message: total + phase breakdown).
Fast and screening rounds have no high-level check phase.

| round | started | ended | elapsed (total) | phase breakdown (loop + sprint / draft / linter / panel / manuscript / check — linter runs in every tier; check is normal-tier only) |

| 1 (fast) | 2026-08-08T08:55:00+08:00 | 2026-08-08T09:05:00+08:00 | 10 min | sprint 1m · draft 1m · attack 3m · manuscript (M, streamline folded) 3m · linter 1m (no check) |
| 1 (normal) | 2026-08-08T09:10:00+08:00 | 2026-08-08T09:24:00+08:00 | 14 min | sprint 1m · draft 1m · attack 3m · manuscript (M, streamline folded) 3m · linter 1m · check 4m |

## Round close (atomic — one pass, at the round-end phase boundary)

The round closes with a **single atomic write** — one section generated in
one pass, never as separate rituals: the timing table above, the dated
subgoals (appended to `question.md`), the decision list, and the ledger
rows this round changed. The user-facing closing message is derived from
this section, not written separately.

- **Round N:** started / ended / elapsed (from the timing table)
- **Sprint:** backlog draws (≤ 3) · settled candidates · overrun discards
- **Subgoals (round N):** <appended to question.md, verbatim>
- **Decision list:** <continue · redirect · repair · park · re-scope · stop
  — one or more, with the routing>
- **Ledger rows:** <panel & check row, claims changes, portfolio re-rank —
  pointer, not copies>
- **Deliverable:** <artifact + version, or negative-value assessment, or
  progress report>

## Open threads / next steps

After an idea-sprint round, the top of the queue is the **sprint backlog**
(≤ 3 draws per round; yields to mid-flight threads).

- **T1** <thread description> — state: active — next: <concrete step>
- **T2** <thread description> — state: stalled <date> — resume: <one line>

## Delivery note (one page, per delivered manuscript)

Models per role and diversity achieved · checks run and verdicts · claims
still `claimed`/`under-review` · computed vs. opined · fetched vs. remembered
· formalization status · tier (screening/fast/normal) and any confidence
downgrade · **canary gate** (detection rate; any miss classified opinion- vs
evidence-kill) · **post-delivery check** (novelty spot-checks run and
findings, numerics audit pass/fail)
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
