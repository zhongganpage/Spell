# Spell — Self-Review Report

**Date:** 2026-08-07
**Scope:** the five Spell documents — `README.md`, `definition.md`, `review-panel.md`, `high-level-check.md`, `protocol.md`.
**Process:** Spell's own review philosophy, run in miniature. Two independent reviewing agents, each with a fresh context, under two lenses:
- **R1 (research-process lens):** persistence, dossier hygiene, exploration, toolkit, open-problem handling over months-to-years.
- **R2 (architecture lens):** independence, adversariality, the 5-agent panel, verification and ledger mechanics.

This document is the synthesis (the "manuscript" step): it ranks the reviewers' ideas, notes where they converge and where they conflict, and proposes concrete changes.

**Meta-caveat (worth stating, and exactly the point of R2's core finding):** the two reviewers ran on the *same base model* with fresh contexts — the "correlated error" regime this report criticizes. Treat the convergences as robust; treat divergences as arguments to weigh, not votes.

---

## 1. Executive summary

Both reviewers independently reach the same verdict: Spell is a **strong verification-and-persistence discipline for a bounded run** ("rough idea in → manuscript out"), and **not yet shaped for its stated target** — genuinely open problems over months to years. Neither reviewer found the design unsound; both found it *structurally incomplete* for the long horizon. The six most serious problems, in order:

1. **No grounding channel.** The panel and the high-level check reason entirely from the model's parametric memory — no computation, no fetched literature, no disproof mandate. The two error classes that waste the most research time, *silent counterexamples* and *phantom novelty*, are the least guarded.
2. **"Independence" is context-freshness, not cognitive diversity.** Five fresh contexts of one model reproduce the same blind spots. Cross-review catches disagreement, never shared error.
3. **The dossier is a log, not a living knowledge state.** Context loss is only partially mitigated; conjectures, obstructions, and "why it fails" insights have no first-class home.
4. **The literature layer gates delivery but is the least reliable component.** A hallucinated `known` verdict kills a working idea; a false `novel` sends months of work down a walked path.
5. **No measurement loop.** Failure modes are documented but never detected; the ledgers accumulate exactly the statistics that would expose them, and nothing reads them.
6. **The author controls the reviewers' diet.** The author prepares the definitions, the cited statements, and which claims get verified — the one real steering channel in the system.

---

## 2. What is genuinely good (keep)

- **Written-record information flow** (R2 §b1). "Only written records", "never the derivation story", fresh context: correctly closes the most important contamination channel — author confidence and false starts.
- **Phase A before exchange** (R2 §b2). Reviewers form their primary view of the draft before seeing each other's — the classic tournament design, correctly placed.
- **The ranking/manuscript split** (R2 §b3). Separate the agent that weighs the record from the agent that writes, with `[ranking deviation]` / `[new in manuscript]` audit hooks. This is the correct answer to the critic-turned-writer collapse.
- **The verification ledger** (R2 §b4, R1 §b2). Real lifecycle, "never used as a premise", bounded repair loops, "verdict outranks confidence" — right rules; the weakness is enforcement and coverage, not design.
- **The persistence machinery** (both). Five-field attempts log, stuck ladder with a floor, R1–R11; M1–M11 is a genuinely good Pólya-style repertoire including the two moves most AI systems skip — M10 (experiments) and M11 (literature, "adapted, not cited").
- **Honest epistemic stance** (both). "Nothing is a certificate" is stated consistently — and, per finding 4 below, applied to proofs but *not* to literature claims.

---

## 3. Findings by theme

### 3.1 Grounding: the biggest gap (R2 #7, #8; R1 C3)

- **Computational grounding:** nothing in the panel phases requires a computation to be re-run, a claim to be tested on examples, or a cited statement checked against its source (R2 #7). The `review-panel.md` Phase A machinery and the `protocol.md` §11 prompt's "hunt for a counterexample" are *mental* exercises. M10 exists but lives in the working loop, not in the panel. **No agent has the job of disproving the main theorem on examples** (R2 #7) — the decisive gap for open problems.
- **Literature grounding:** the high-level check's novelty search ("searched against the literature", `high-level-check.md`) is undefined: no retrieval mechanism, no database, no requirement to fetch and read anything — in practice it is parametric memory, where hallucinated citations live (R1 C3.1; R2 #7). A false `known` routes real work to "reformulate" and discards it; a false `novel` delivers a duplicate of the literature after months (both).
- **Sufficiency is an LLM opinion with no burden of proof:** "within reach of known tools" is one of the hardest judgments in mathematics, yet a `pass` requires no supporting argument and no bottleneck certificate naming the weakest sub-claim the whole approach hinges on (R2 #8). R1 C1 adds that the `sufficient` gate and the Phase A "correct in shape" item presume a theorem-shaped draft, which a progress report is not.

### 3.2 Independence vs. diversity (R2 #1, #2; R1 C4.5)

- Same-model fresh contexts share weights, priors, and blind spots; three "independent" reviewers of one model are a weak form of one reviewer (R2 #1). Cross-review corrects disagreement, not shared error — and Phase C happens *after* the exchange, so it can convert correlated error into mutually reinforcing agreement (R2 #2).
- The panel ledger records "panel agents used" but nothing about *which models* ran or whether diversity was achieved — a same-model run is indistinguishable from a diverse one, with no confidence downgrade (R2 #1).
- R1 C4.5 adds the cheap hedge: when only one model is available, differentiate *roles* (counterexample-hunter, step-by-step validator, architecture critic) instead of three identical contexts.

### 3.3 The dossier is an archive, not memory (R1 C2 — called "the deepest weakness"; R2 #9, #10)

- **Five lines per session is aggressively lossy** (R1 C2.1). The expensive knowledge is *why* something fails; R3 prevents re-attempting recorded dead ends, not re-deriving the insight.
- **No living synthesis layer** (R1 C2.2). Append-only guarantees integrity but forbids the maintained "current knowledge" document a 6-month gap demands. "Read the dossier in full" becomes unenforceable when the dossier outgrows the context window — the append-only rule and the readability requirement are in unresolvable conflict as written.
- **Conjectures have no home** (R1 C2.3). The ledger's only status `claimed` requires "the location of its proof" — a conjecture by definition has no proof, and R5 generates conjectures constantly. Conjectures are the primary currency of open-problem research and the worst-supported artifact.
- **No obstruction register** (R1 C2.4). `implies` fields record single-failure lessons; nothing aggregates "every attack dies at the commutation obstruction" — the metacognitive step that converts dead ends into a research plan is absent.
- **No dependency links** (R1 C2.5; R2 #9). "Never used as a premise" is enforced by fiat; nothing tracks dependents, so a `counterexample` verdict leaves dependents sitting `claimed` silently. "Never reuse a stale fact" requires knowing where a fact is used.

### 3.4 The author controls the reviewers' diet (R2 #5 — "the single most exploitable information asymmetry")

The author prepares which definitions, how cited results are stated, and which claims get verified (`review-panel.md` "Inputs and outputs"; `protocol.md` §11). Omitting a cited theorem's hypotheses, or restating it loosely, is a classic silent failure — and the panel has no access to the originals. The contest path lets the author frame the dispute. Mostly unconscious steering, but the party being reviewed controls what the reviewers read.

### 3.5 Adversariality is invoked, not measured (R2 #3, #9, #10; R1 C6.4, C7)

- **Ritualism is manufactured by the format** (R2 #3; R1 C6.4): Phase A mandates a gap hunt, so an agent with an empty "gap hunt" looks like a failed reviewer; Phase F then forces M to dispose of every manufactured objection. The rejection rule covers ritualistic *rebuttals* only — nothing rejects ritualistic *criticisms*.
- **Groupthink is rewarded** (R2 #2): the ranking agent is told to rank the "persuasive" ideas — a consensus detector, and consensus is what groupthink manufactures. Findings from independent Phase A are not distinguished from later echoes at ranking time; that provenance exists in the phase structure and is thrown away.
- **No statistics** (R2 #10): the failure modes are a documentation section. The data to detect them (gap tags later withdrawn, `pass` manuscripts later accumulating `rejected` claims, per-agent false-positive rates, agreement rates) sits in ledgers nobody reads. "Recording is mandatory" with no reader for the records is the largest unforced error (R2 #10).
- **R1/R2 gaming** (R1 C7): "concrete next step" is unenforced, and the success criterion ("dossier strictly more informative") is self-assessed by the one agent forbidden from grading itself.

### 3.6 The panel is built for manuscripts, not for research advice (R1 C6; R2 #6, #11)

- **Panel insight is not routed into the loop** (R1 C6.2): Phase A "suggested next attacks" and R's ranking — the best research advice the system generates — are written to the manuscript agent, and nothing requires the working loop to consume them.
- **Reviewers can't advise on exploration** (R1 C6.3): independence from the derivation means the panel cannot say "your failed attempt almost proves X" — the most valuable thing a collaborator says. Fix: drafts carry a "what was tried and failed" section.
- **Co-adaptation + discarded knowledge across rounds** (R2 #6): over revision cycles the author learns the panel's taste (a "critique equilibrium"), while freshness rules throw away the accumulated critical content each round. Spell resolves independence-vs-continuity *always* in favor of freshness — wrong for the open-problem regime.
- **Adversariality only at the draft boundary** (R2 #11): the months of actual research run as one self-recording, self-grading agent; the agent that just failed on a strategy chooses the next strategy, inheriting its own confirmation bias. The strongest guarantee in the system applies only at the end of the research.

### 3.7 Cost model does not scale to a year (R1 C4, C5; R2 #8)

- **Verification is linear and untiered** (R1 C4.1–4.2): every claim gets a full fresh session; code-checkable computations get the same treatment as the central lemma; no delta-only mode. Cost either explodes or compliance quietly fails — which then silently breaks "nothing may build on it".
- **The floor feeds the review queue** (R1 C4.4): R11 trial proofs flood the ledger with low-value artifacts.
- **Threads are a FIFO queue, not a portfolio** (R1 C5): no periodic re-ranking by expected value, and parked threads have no revival triggers ("re-examine when a paper on X appears").

### 3.8 No human channel, no refinement path (R1 C8)

- The human leaves the loop after the output-form question; no user-notes channel, no mechanism to inject literature or redirect priorities. On a year-long problem this excludes the most reliable source of new information.
- The locked problem statement cannot refine: on any real campaign the conjecture gets refuted and the true theorem is a variant, but there is no formal "problem revision" event — refinement forks an orphaned dossier or silently violates the lock.
- Minor: the once-only output-form question has no revision path; "statements only" of cited results also weakens claim verification, not just panel review (R1 C9).

### 3.9 Ceremony exceeds certificate (R2 #12)

Five agents, six phases, rebuttals, rankings, ledgers, checks — and the deliverable is a polished manuscript with no attached metadata: which models ran, whether diversity was achieved, which claims remain `claimed`/`under-review`, what was computed vs. opined, what was fetched vs. remembered. The distinction between "reviewed by N LLMs, unverified computationally" and "proved" is the entire epistemic content of the deliverable, and it is handed over as an implicit assumption.

---

## 4. Points of tension between the reviews

These are real disagreements, not duplication; each needs a decision.

1. **The minimum-output floor (R11).** R1 (improvement #12) proposes down-tuning: when no direction remains, the honest output is the next step plus one line of "why nothing is ripe", and the trial-proof floor becomes optional — protecting the dossier's signal-to-noise and relieving the review queue (R1 C4.4, C7.2). This conflicts with the persistence protocol's core philosophy (R11 is the floor that "no amount of difficulty waives"). *Middle path:* keep the floor mandatory but tiny (≤10 lines), and keep floor artifacts out of the verification queue (they enter as `claimed` but with a `floor` tag that exempts them from full review).
2. **The delivery contract.** R1 (C1) argues the campaign's real outputs — partial results, refuted approaches, obstructions — have no exit, so the contract should allow a non-manuscript deliverable. R2 does not address this but its delivery-note proposal (#12) would apply to any exit. *Decision:* keep "manuscript only" for the theorem, but add a documented "research report" exit so the loop has a legitimate terminal state that is not over-claiming.
3. **How much process to add.** R2's improvements are mostly *more machinery* (auditors, trace-checkers, canary panels); R1's highest-ranked items are mostly *restructuring the dossier*. Both are needed, but the order matters: knowledge-state restructuring (R1 #1–2) is cheap and changes the system's character most; the auditor machinery (R2 #2, #3) is where the effort should go second.

---

## 5. Prioritized improvements (merged from both reviews)

Tier 1 — implement before trusting Spell on a genuinely open problem:

1. **Ground the machinery in external oracles** (R2 #1/#7; R1 C3). Give the panel a *computational adversary* role: required to test the draft's key claims on concrete instances and edge cases by code/CAS, run logs attached as written records, with an explicit disproof mandate on the main theorem itself. Require novelty/sufficiency verdicts to cite sources actually fetched and read; an unverifiable citation forces at most `conditional`; attach a search-coverage statement (sources, queries, years, negative results). **This attacks the two error classes that waste the most research time.**
2. **Make "independence" honest** (R2 #1; R1 C4.5). Record which models/backends ran per role in the ledger; when only one model is available, differentiate reviewer roles (counterexample-hunter, step-validator, architecture critic) and mark the run as role-diverse rather than model-diverse, with a confidence downgrade on the outputs.
3. **Turn the dossier into a living knowledge state** (R1 #1–2; R2 #9/#10). Add a **Knowledge State** section — *rewritten at the end of every session, not append-only* — holding: best established results (pointers into the ledger), active conjectures with evidence and age (Conjectures registry: `active`/`supported`/`refuted`/`parked-until`), identified obstructions and the threads they block (Obstructions register), the champion draft pointer, and a one-paragraph "where the project stands". Fresh sessions load Knowledge State first. Add dependency backlinks with invalidation propagation (`counterexample` flags dependents `affected`).
4. **Take the author out of the reviewers' diet** (R2 #4). Panel inputs (definitions, cited statements) copied verbatim from the dossier's locked sections; any restatement flagged `[restated]` and rejectable; the §11 reviewer prompt filled by the harness from the dossier, and reviewer verdicts logged to the ledger directly by the reviewer session.
5. **Add a measurement loop** (R2 #2/#10; R1 C8.1). A periodic panel auditor reads both ledgers and computes: Phase A `gap` tags later contradicted or withdrawn (ritualism), `pass` manuscripts later accumulating `rejected` claims (gate leakage), per-reviewer false-positive rates, Phase A agreement rates. Feed results back as standing instructions to future panels and as confidence budgets on delivered manuscripts. Periodically run a *canary panel* on a seeded known-false claim to measure detection.

Tier 2 — hardening:

6. **Provenance-tag findings** (R2 #5): tag every finding as `Phase A (independent)` or echoed; R ranks independent agreement above post-exchange echo and unique catches above consensus echoes.
7. **R and M accountability** (R2 #3): R emits a *tension list* (every material disagreement in the record and how it was resolved); M emits a traceability table (assertion → record artifact + item) verified by a cheap fresh-context trace-checker; allow M one written question round to R; optionally run ranking twice (soundness-first vs. promise-first) with reconciliation.
8. **Continuity across rounds** (R2 #6): after a failed manuscript, the next draft carries a "response to the panel" section; the next panel receives the previous rounds' critique list and judges whether the responses actually resolve them.
9. **Ledger upgrades** (R2 #8/#9; R1 C4): add a `disputed` state and a third-opinion path (two disagreeing sessions → third fresh session with both verdicts verbatim); the two-round bound counts across sessions for the same claim; track claim dependencies.
10. **Bottleneck certificates + early triage** (R2 #8): every high-level check — `pass` included — names the single weakest sub-claim the approach hinges on and routes it into the open threads; make the novelty triage mandatory on the *first* draft, not optional.
11. **A second mind inside the working loop** (R2 #11): at stuck-ladder rungs ≥ 3, a fresh-context *strategy critic* reads the attempts log, challenges the next-step choice and the failure diagnosis, and files its objection in the dossier.
12. **Tiered verification** (R1 #4): code-checkable computations get a cheap scripted check instead of a full session; delta-only verification relative to accepted claims; floor artifacts tagged to stay out of the review queue.
13. **Thread portfolio management** (R1 #5): periodic re-ranking of active threads by expected value; every parked thread carries a "re-examine when <event>" trigger tied to the literature map as a watch list.
14. **Route panel insight back into the loop** (R1 #6): Phase A "suggested next attacks" and R's ranking are appended to "Open threads / next steps" verbatim; drafts must include a "what was tried and failed" section; escalation threshold after N consecutive failed manuscript cycles (switch to a research-review mode on the exploration record).
15. **Human touchpoints** (R1 #8): a `user-notes` dossier section; a required periodic review with the user of parked threads and portfolio rankings; a formal problem-refinement path (dated amendments to the locked statement with a change record).
16. **Delivery note** (R2 #10): one page attached to every delivered manuscript — models per role and diversity achieved, checks run and verdicts, claims still `claimed`/`under-review`, computed vs. opined, fetched vs. remembered. Operationalizes "nothing is a certificate" at the moment of delivery.

---

## 6. Open decisions for the user

1. **Delivery contract:** strict "manuscript only", or add a partial-result / research-report exit so long campaigns have an honest terminal state (tension 2)?
2. **R11 floor:** keep mandatory, soften to a tiny artifact, or make optional (tension 1)?
3. **Implementation scope:** adopt all of Tier 1, or start with the cheap high-leverage items (Knowledge State + Conjectures/Obstructions registries, verbatim reviewer inputs, ledger diversity reporting)?
4. **Grounding feasibility:** the computational-adversary role requires the harness to run code/CAS — acceptable on this machine, or should it be a manual/user-assisted step?

The two reviewers' final judgments, in their own words: *"The first five items are the difference between Spell working on open problems and Spell working only on bounded problems"* (R1). *"A strong scaffold that will reliably produce plausible manuscripts; it needs grounding, diversity, measurement, and cross-run continuity before it can be trusted to steer research"* (R2).

---

## 7. Addendum — user decisions (2026-08-07)

After this report, the user made two decisions that modify the design:

1. **Run envelope.** Spell runs are time-boxed to **`RUN_LENGTH`** of agent
   work per run, chosen at project start; every run ends
   with a deliverable (draft, manuscript, or progress report) and a decision
   list for the user. Spell never operates autonomously across days or
   months. This resolves tension 1 and item 15
   (human touchpoints) at the level of the operating mode: the user is in
   the loop between runs by construction.
2. **Exterior panel agent.** One of the three panel reviewers is replaced by
   an **exterior agent (X)** called through another company's agent API,
   configured at project start (provider/model + API access, credentials
   kept out of the dossier). This implements improvement 2 structurally —
   genuine model diversity instead of same-model fresh contexts — with the
   `reduced diversity` fallback when X is unavailable.

Implementation status: reflected in `definition.md`, `review-panel.md`,
`protocol.md`, and `README.md`. Still open from the report: the rest of
Tier 1 (grounding, measurement loop, living Knowledge State in the dossier)
and the open decisions in §6.
