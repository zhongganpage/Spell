# Spell — Architecture Update Plan

**Status:** LOCKED — approved 2026-08-08T17:09:00+08:00 (rev. 2, nine review fixes applied).
**Date:** 2026-08-08
**Sources synthesized:**
- Theoretical review of Spell's architecture (discovery / emergence focus), 2026-08-08.
- Post-mortem of the LDP-homog run, rounds 1–4 (`spell-architecture-review.txt`): convergence & hygiene failures.
- Post-mortem of the nonexistence-L2 run, rounds 1–7 (`spell-architecture-review.md`): under-use of existing tiers, numerical guardrail, depth commitment.
- The existing `reference/self-review.md` (2026-08-07) and the decisions taken since.

Each item below cites its source (`TH` = theoretical review, `LDP` = LDP-homog post-mortem, `L2` = L2 post-mortem, `SR` = self-review.md). Items with two or more independent sources are the reliable core and are prioritized.

---

## 0. Design principles of this update

1. **Nothing delivered may rest on a non-accepted premise.** This invariant does not move. Everything about how ideas live and develop is free to change.
2. **Opinions rank; only demonstrated counterexamples kill.** "Known", "beyond reach", "misdirected" are opinions — they may demote and attach revival triggers, never terminate. (The user may always park a thread by their own decision; the no-kill rule binds the protocol's mechanisms, not the user.)
3. **Deterministic checks beat model checks for mechanical error classes.** Units, citations, brackets, numerical exact-case reproductions are linter/script territory, not reviewer territory.
4. **Round type selects pipeline weight.** A screening round, a negative round, a repair round, and a manuscript-bound round do not all run the 5-agent panel.
5. **The loop must converge monotonically.** Artifact size non-growing, open-condition count non-increasing, per-claim verification depth non-decreasing. Violation of any of the three is a protocol alarm.
6. **The user steers at verdicts.** `conditional` / `fail` / negative-outcome routing is a user decision point (repair / park / re-scope), never an automatic continue.
7. **Ground truth is un-overturnable.** A formalized lemma (Lean 4) is the only check the next round's gate cannot overturn; load-bearing claims should be anchored to it where feasible.
8. **Budget is conserved.** The savings from tier selection fund the new phases (sprint, linter, promoter, negative-value assessment); total per-round agent budget does not grow.

---

## 1. Phase 1 — Governance & hygiene (highest leverage, cheapest)

The empirical reviews' top findings, implemented as policy. Ships first; touches routing and one new phase.

### 1.1 Tier-selection policy (L2 §4.2, LDP #6; TH-G)
**Change.** Replace the binary normal/fast mode question with a three-tier choice, and add defaults that actually bind:
- **Screening** — claim-reviewer format (prompts.md §1) on every claim nominated for the verification pipeline. Default for negative/exploratory rounds.
- **Fast** — existing 2-agent A/B loop. Default for negative rounds and repair rounds; repair rounds escalate to normal only when a listed condition is load-bearing.
- **Normal** — the 5-agent panel. Required only for load-bearing or manuscript-bound claims.
The orchestrator proposes a tier from the round type at round start; the user may override; the choice is recorded in the dossier (extend invariant 10).
**Files.** `core.md` (Round mode §), `reference/protocol.md` §2, `reference/definition.md` ("fast mode", new "tier" entry).
**Acceptance.** No round runs the full panel when its round type maps to a cheaper tier.

### 1.2 Claim-reviewer screening as the default gate (L2 §2.2, §4.1)
**Change.** Every claim **nominated for the verification pipeline** (i.e., entering `claimed`/`under-review`) runs the prompts.md §1 claim-reviewer format *before* any panel decision. Only screening-passing claims (or load-bearing ones explicitly nominated) reach the panel. A claim that fails screening returns to the loop as a repair task — the existing ≤2-repair rule applies. **Speculative and promising ideas (3.1) are exempt** — they are not yet claims in the pipeline. *Dependency: 3.1's status labels ship with this item (see §6).*
**Files.** `reference/protocol.md` §8, `modules/prompts.md` §1 (keep), `modules/dossier-template.md` (ledger gains a `screening` row).
**Acceptance.** Panel cost per round drops; screening verdicts visible in the claims ledger.

### 1.3 Hygiene linter phase (LDP #2; L2 2.1; TH-G)
**Change.** New mandatory phase — a **deterministic** mechanical pass by one cheap agent/script, run **before any review phase and before delivery in all tiers**:
1. every displayed equation checked for dimensional/normalization consistency (factor-2, brackets, per-edge vs per-volume, tilt factors);
2. every citation checked against fetched records (locators present, not corrupted);
3. every bracket/range/constant checked against the manuscript's own tables and the notation lock.
**Files.** `reference/protocol.md` (new §, "The hygiene linter"), `modules/prompts.md` (new prompt), `reference/review-panel.md` (pipeline position).
**Acceptance.** Would-have-caught rate ≥ 70% of the mechanical defect classes seen in LDP rounds 1–4 (measured retroactively on those reports).

### 1.4 X diversity enforcement + orchestration checklist (LDP #1, #8; L2 2.3; TH)
**Change.**
- At startup, verify `X_MODEL` is not the same provider family as the internal harness. If it is (or X is unavailable), the run is auto-labeled `reduced diversity` with a confidence downgrade — one-line check, non-negotiable.
- Orchestration checklist: A1 + A2 + X launched in parallel; every artifact's existence confirmed before the next phase; X's absence (incl. "never launched", L2 round 5) recorded as a ledger event.
**Files.** `reference/review-panel.md` (exterior agent), `reference/protocol.md` §9, `modules/providers.md`.
**Acceptance.** A same-family or absent X is always flagged; no silent same-model panel.

### 1.5 Numerics guardrail (L2 2.1; LDP #4)
**Change.** Every numerical claim carries:
1. an **exact-case reproduction** (a ≡ const, d=1, a 2×2 model) embedded in the script — the a ≡ const check alone would have exposed the L2 interpolant bug;
2. **two independent scripts** for load-bearing numbers;
3. **role separation** — the agent that builds the theory is not the agent that writes the verification scripts.
Shared, versioned norms/solver file that all agents must cite (notation lock enforced, not just claimed).
**Files.** `reference/protocol.md` §4–5, `modules/toolkit.md` (M10 amendment), `modules/dossier-template.md` (numerics register).
**Acceptance.** No load-bearing number rests on a single self-written script.

### 1.6 GAP-owner discipline (L2 2.6; TH-F)
**Change.** One thread owns one GAP across multiple rounds until the stuck-ladder floor is reached. Targets change only at a floor verdict; the portfolio (Phase 3, 3.5) reserves the champion share for that owner.
**Files.** `reference/protocol.md` §6 (stuck ladder), `modules/toolkit.md`.
**Acceptance.** A GAP is attacked from ≥ 2 angles across rounds before abandonment; each angle recorded.

---

## 2. Phase 2 — Convergence mechanics

### 2.1 Non-terminal gates + revival triggers + fragments rule (TH-B; LDP #1)
**Change.**
- High-level check `fail` and panel "misdirected" **demote** (portfolio rank down) and attach **revival triggers** ("re-examine when <event>", wired to the literature watch list and new dossier evidence). **No verdict auto-parks a thread** — the old automatic "reformulate or park" routing is removed; parking is an explicit user decision (2.3), and even then the fragments rule applies.
- The only terminal state for an idea is a **demonstrated counterexample with a reproducible computation**.
- **Fragments rule:** every terminal verdict (counterexample or evidence-backed rejection) must deposit the maximal true subcase, the obstruction, and the closest technique into the wild-ideas register with revival triggers.
**Files.** `reference/high-level-check.md` (decision routing), `reference/protocol.md` §8, `modules/dossier-template.md`.
**Acceptance.** No thread is terminally parked on an opinion; every death leaves a reusable fragment.

### 2.2 Negative-value assessment (L2 2.7; TH-H)
**Change.** New deliverable form for negative rounds: a **negative-value assessment** — did we truly rule out a direction (with evidence: exact cases, fetched literature, certified computation), or did the tool just fail? It is produced by a **dedicated cheap agent that runs in all tiers** (fast and screening rounds do not skip it — it *replaces* the high-level check for negative outcomes, it is not a form of it). Its job is to **assess** (not certify) the evidence quality of the rule-out. A rule-out with assessed evidence is a first-class deliverable, recorded in the fragments register.
**Files.** `reference/definition.md` (new artifact), `reference/high-level-check.md`, `reference/protocol.md` §2.
**Acceptance.** Negative rounds deliver assessed rule-outs instead of forced manuscripts.

### 2.3 Repair-scoped rounds with user gating (LDP #3, #5; L2 §4.3; TH-B)
**Change.** `conditional` verdicts are always a user decision point: **repair** (scope frozen — fix the listed conditions only, no new conjectures, re-check only those at the end) / **park** / **re-scope**. Repair rounds run the cheapest tier that can check the listed conditions; the artifact is *not* re-derived. Parking is a user decision, with fragments + revival triggers recorded.
**Files.** `reference/protocol.md` §2, `reference/high-level-check.md`, `reference/definition.md` ("round").
**Acceptance.** Conditional verdicts never auto-continue into a full new round.

### 2.4 Depth-of-verification escalation (LDP #1)
**Change.** Two separate axes:
- **Artifact hygiene** — the linter (1.3) runs every round, every tier.
- **Claim verification depth** — per claim, non-decreasing across rounds: `screening → panel → formalization`. A claim that survives round N is not re-checked at the same depth in round N+1; the ledger's `depth` column tracks claim-level depth. The gate's verdict is *descriptive routing* (to the next depth or to the user), never a certificate.
**Files.** `reference/protocol.md` §8, `modules/dossier-template.md` (claims ledger gains a `depth` column), `reference/high-level-check.md`.
**Acceptance.** "Passed in round N" is followed by deeper checking, not repetition; no verdict treated as certification.

### 2.5 Surgical M + folded streamline (LDP #5; L2 2.5)
**Change.** M patches only the sections the record changed (diff-scoped manuscript) instead of rewriting the full document; the streamline step folds into M; ledgers grow as appendices rather than being rewritten. Artifact size is monotone non-growing per round (a protocol alarm otherwise). Repair agents `resume` the panel agent (keeps context) instead of re-reading everything fresh.
**Files.** `reference/review-panel.md` Phase F, `reference/protocol.md` §2 (streamline), `reference/definition.md` ("manuscript").
**Acceptance.** Manuscript page counts do not grow across rounds without a recorded reason; repair costs drop.

### 2.6 Knowledge State populated + dossier split + question.md hygiene (L2 2.4, 2.8; SR 3.3)
**Change.**
- The Knowledge State section (conjectures registry, obstructions register, champion pointer) is **populated and maintained** as the navigation index, as designed but never done (L2 2.4).
- Split the dossier by section once it exceeds ~30 KB: `claims-ledger.md`, `attempts-log.md`, `version-inventory.md`, top-level dossier stays as the navigation index. Fixes concurrent-edit conflicts (`old_string not found`).
- question.md stays minimal: statement + dated subgoal list only; analysis lives in the dossier.
**Files.** `modules/dossier-template.md`, `reference/protocol.md` §3.
**Acceptance.** No concurrent-edit failures; Knowledge State is the first thing a fresh session reads.

### 2.7 Adaptive phase depth (LDP #6)
**Change.** When the three Phase-A review verdicts are within one category (all accept / all repairable / all misdirected), the full B exchange → C cross-review → D rebuttal sequence collapses into a **single combined response exchange** (each reviewer answers the others once). When verdicts genuinely conflict, the full B/C/D runs. **Records are never merged** — every position change stays verbatim (the retraction discipline is the architecture's best asset and must survive the merge). LDP estimate: ~6 agents/round → ~3 on agreeing panels.
**Files.** `reference/review-panel.md` (phases B–D), `modules/prompts.md`.
**Acceptance.** Agreeing panels cost ~half the agents; conflicting panels keep the full sequence.

---

## 3. Phase 3 — Discovery & retention (the emergence track)

### 3.1 Claim lifecycle: `speculative` → `promising` (TH-A)
**Change.** Extend the lifecycle:
```
speculative → promising → claimed → under-review → accepted | rejected | counterexample
```
- `speculative` — wild-ideas register; exempt from the verification ledger and from panel attack; leaves only by nomination.
- `promising` — nominated for development; **may be built upon heuristically** in exploration, every use labeled `heuristic use of <claim vN>`; never a premise in any delivered artifact. Promising-based heuristics may appear in drafts (labeled); they never appear in delivered artifacts or in user-facing reporting of established results.
`claimed` and beyond unchanged. This legalizes development-through-use — the mechanism by which embryonic ideas mature — while the delivery gate stays fixed.
**Files.** `reference/protocol.md` §8, `modules/dossier-template.md`, `modules/prompts.md` §1.
**Acceptance.** A `promising` claim can be pushed forward in the loop without entering the review queue; no delivered artifact cites one.

### 3.2 Idea sprint (cheap, parallel) + recombination agent (TH-C, corrected by L2 2.6)
**Change.** At round start, a bounded (≤ 10% budget) parallel sprint: 3–5 explorer agents (flash models fine) — specialize/edge-case miner, reformulation hunter (M5), analogy/transfer agent (reads obstructions register + literature map), wildcard — each returning 3–5 candidate attacks with the *cheapest discriminating test*. Plus a **recombination agent** pairing unrelated dossier entries (two reformulations; an obstruction + a partial result; an idea + a technique that worked elsewhere). All candidates, including discarded ones, go to the wild-ideas register with revival triggers. **The sprint feeds the GAP-owner; it does not open new target queues** (L2 2.6). **The wild-ideas register is bounded:** ≤ 15 *active* wild ideas; the rest are archived with their triggers and re-checked only when a trigger fires or the idea-yield ranking promotes them. The sprint's analogy/transfer agent evaluates revival triggers each round.
**Files.** `reference/protocol.md` §2 (new phase), `modules/prompts.md` (new prompts), `modules/dossier-template.md` (wild-ideas register).
**Acceptance.** Each round generates ≥ N new candidate attacks at flash-model cost; champion threads keep their budget; the register stays bounded.

### 3.3 Promoter role (TH-D)
**Change.** A fresh-context agent alongside the panel whose job is to *push the champion idea as far as it goes*: the strongest honest version, the maximal true fragment, exactly where it breaks, and what the break implies the true statement must be. Mirror of the attacker; the role most likely to find the proof. Its outputs enter the ledger as `promising`/`claimed` — its own work never grades itself (invariant 1).
**Files.** `reference/review-panel.md` (new role), `modules/prompts.md` (new prompt).
**Acceptance.** Every manuscript-bound round includes a "nearest true version" note from the promoter.

### 3.4 R re-mandate + idea scoreboard + provenance tags (TH-E; SR §5)
**Change.** R ranks "promising-for-the-goal", not "persuasive" (persuasiveness is a consensus detector). Findings carry provenance tags (`Phase A independent` vs. echoed); independent agreement and unique catches outrank consensus echoes. The panel emits a per-round **idea scoreboard**: each candidate rated promise × reach ÷ cost. M may carry a `[speculative developments]` appendix whose entries are flagged, not forced through the "every gap resolved" obligation (modifies Phase F obligation 2 for speculative content only).
**Files.** `reference/review-panel.md` Phase E/F, `modules/prompts.md` §3–4.
**Acceptance.** Unique independent catches rank above echoed consensus; speculative content is flagged, not resolved-by-force.

### 3.5 Portfolio allocation + idea-yield metric (TH-F; L2 2.6)
**Change.** Open threads become a ranked population with a fixed budget split (70% champion / 20% runners-up / 10% wild). Re-ranked each round by the **idea-yield metric** — accepted claims + promoted ideas + new subgoals per unit cost per thread — computed from the ledgers. GAP-owner threads hold the champion share until the ladder floor.
**Files.** `reference/protocol.md` §3, `modules/dossier-template.md` (thread table), `modules/prompts.md` §7 (auditor).
**Acceptance.** Budget allocation is visible in the dossier; champion share changes only on floor verdicts or yield ranking.

### 3.6 Auditor: premature-kills metric + mandatory canary (TH; SR §5; L2 under-use)
**Change.** The auditor (invariant 8) additionally computes **premature kills** — ideas rejected/`fail`ed/parked that later proved right (later round, literature, or user). A single premature kill is a protocol alarm; the rate distinguishes **opinion-kills** (verdict-based — process fixable) from **evidence-kills** (counterexample-based — correct behavior). The auditor also **evaluates revival triggers** at each run and computes the idea-yield metric. One canary panel (seeded known-false claim) becomes mandatory before the first normal-mode delivery on a new project.
**Files.** `modules/prompts.md` §7, `reference/protocol.md` §8, `reference/self-review.md`.
**Acceptance.** Premature-kill count (by kill type) reported every run; a canary has run per project; triggers are checked.

### 3.7 Formalization anchor (LDP #7; L2 under-use; TH-B)
**Change.** At least one load-bearing lemma per project is delegated for Lean 4/mathlib formalization (M12, prompts.md §6) — the only un-overturnable check. Self-contained packages (e.g., LDP's d=1 T1–T4) are the default first target. The delegation is bounded by the run envelope like any M12 task.
**Files.** `reference/protocol.md` §5.1, `reference/definition.md`, `modules/prompts.md` §6.
**Acceptance.** Every project has ≥ 1 machine-checked lemma or a recorded reason why not.

---

## 4. Mechanics fixes (from both runs)

- **M1 — Live timestamps (LDP #7).** Phase timestamps recorded at the moment (`date`), never reconstructed from notification times. Enforced by the orchestrator; violation is a protocol alarm.
- **M2 — X pipeline (LDP #8; L2 2.3).** Writable output dir or standardized markers for `codex exec`; summary+inline-reports prompt to stop 50–100 KB re-reads; default to kimi/k2.7/api (no sandbox) unless codex is required.
- **M3 — Agent resume for repairs (L2 2.5).** Repairs resume the panel/author agent (keeps context) instead of re-reading everything fresh.
- **M4 — Sync the skill package.** After implementation, `skills/spell/` in the repo and `~/.kimi-code/skills/spell/` (installed copy) are updated together; version bump in the frontmatter description.

---

## 5. File-by-file change map

| File | Changes |
|---|---|
| `core.md` / `skills/spell/SKILL.md` | pipeline + tiers (§1.1), invariant rules (screening, linter, non-terminal gates), vocabulary (negative-value assessment, wild ideas, promising) |
| `reference/protocol.md` | §2 pipeline & round types; §3 dossier split + Knowledge State + portfolio; §5.1 delegation + formalization; §6 GAP-owner; §8 lifecycle + screening + escalation; §9 mechanics; new "hygiene linter" |
| `reference/review-panel.md` | Phase E R re-mandate; Phase F surgical M; new promoter role; idea scoreboard; provenance tags; adaptive phases B–D (2.7) |
| `reference/high-level-check.md` | non-terminal routing + revival triggers; negative-value assessment; escalation of depth |
| `reference/definition.md` | new artifacts (negative-value assessment, wild ideas, promising), tier definition |
| `modules/prompts.md` | new: linter, promoter, sprint explorers ×4, recombination, negative-value assessment; updated: §1 screening, §3 ranking, §7 auditor |
| `modules/dossier-template.md` | wild-ideas register, promising claims, numerics register, idea-yield table, dossier split, Knowledge State as index |
| `modules/toolkit.md` | M10 numerics guardrail amendment; GAP-owner note |
| `reference/self-review.md` | append this plan and the two post-mortems as the 2026-08-08 follow-up |
| `README.md` | design principles (§0), updated pipeline diagram |

## 6. Sequencing & validation

**Order:** Phase 1 (policy + hygiene) → Phase 2 (convergence) → Phase 3 (discovery) → mechanics.
Rationale: Phase 1 cuts cost and error classes immediately and touches routing only; Phase 2 makes the loop monotone; Phase 3 adds the discovery machinery on top of a stable loop.
**Dependency note:** item 1.2 (screening) depends on 3.1's *status labels* (`speculative`/`promising` exemptions); the labels ship with Phase 1, the full lifecycle machinery (sprint, promotion, registers) with Phase 3. **Budget:** tier savings fund the sprint/linter/promoter/assessment; per-round agent budget conserved (principle 8).

**Validation on the next real runs** (LDP-homog round 5 / L2 round 8, or a fresh project):
1. Panel cost per round down (tiers working).
2. Mechanical defect classes caught before the panel (linter).
3. Artifact size monotone non-growing; condition count non-increasing; verification depth non-decreasing.
4. Premature kills: 0 opinion-kills.
5. Rule-outs assessed (negative-value assessments) instead of forced manuscripts.
6. ≥ 1 formalized lemma or recorded reason.

**Rollback:** each item is a scoped text change to one or two files; any single item can be reverted without touching the rest.

---

## 7. Open decisions for the user

1. **Tier defaults.** Auto-select the tier from round type (recommended), or always ask? Auto-select with override is recommended.
2. **Canary frequency.** One canary per project (recommended) or per N rounds?
3. **Formalization requirement.** Mandatory one lemma per project, or best-effort when a self-contained package exists?
4. **Dossier split threshold.** Split at ~30 KB, or only on first concurrent-edit failure?
5. **Implementation scope.** Phase 1 + mechanics first, then 2, then 3 — or all at once? *(resolved 2026-08-08: full implementation, all phases, this session)*
