# Spell — Ready-to-Paste Prompts

Load this module when a phase starts. Each prompt is pasted into a fresh
session (internal agents) or passed to X (`codex exec "<prompt>"` /
provider API). Attach the inputs listed under each.

**Delivery contract (the orchestrator applies this to every assembled
prompt).** Spawn the agent with the harness's explicit background/async
spawn — `run_in_background=true` in an `Agent`-tool harness (Kimi Code,
Claude Code) — never the blocking foreground default: the agent runs
hidden, the orchestrator never blocks on it, and the artifact is collected
when it arrives. Add an explicit output path to the prompt, and require the
agent to write its artifact there and confirm the write in its final
message. If the agent's environment is read-only or the write fails (a
read-only sub-agent type, a sandboxed `codex exec`), the agent must instead
include the complete artifact text in its final message; the orchestrator
persists it verbatim at the assigned path and marks the record
`recovered from agent output`. Verify the file exists before the next phase
starts (Invariant rule 9; `protocol.md` §9).

## 1. Claim reviewer — verification ledger

**Screening gate (tier screening; plan §1.2).** This format is the default
gate for every claim entering the verification pipeline (`claimed` /
`under-review`): it runs before any panel decision — in the screening tier
and wherever a claim is nominated. Only screening-passing claims, or
load-bearing ones explicitly nominated, reach the panel; a claim that fails
screening returns to the loop as a repair task (the existing ≤2-repair rule
applies). `speculative` and `promising` ideas are exempt — they are not yet
claims in the pipeline (`reference/definition.md`, "promising").

```text
Independently review the mathematical claim and proof below. You have not
seen how the author derived them; do not assume the author's reasoning is
sound. Judge only what is written.

A. CLAIM — Is the statement precise, complete in its hypotheses,
   well-defined in its terms, and exactly what it claims to be (not weaker,
   not a different claim)?
B. PROOF — Does every step follow from the results it cites, and only from
   them? Do the cited results' hypotheses hold at each point of use? Is any
   step silently assuming something unproved? Do the quantifiers match the
   claim?
C. BOUNDARY — Check degenerate and edge cases, check the smallest nontrivial
   case, and hunt for a counterexample to the claim or to a single step.

Verdict, with reasons: accepted | rejected (with specific repair targets) |
gaps found (non-blocking notes).

--- CLAIM (exactly as stated) ---
<claim text>

--- DEFINITIONS (exactly) ---
<definitions used>

--- CITED RESULTS (statements only, not proofs) ---
<cited results>

--- PROOF (exactly as written) ---
<proof text>
```

## 2. Panel reviewer — Phase A (for A1, A2, X/A3)

```text
You are the sharpest reviewer in this room; no nuance, gap, or promising
idea will escape you. You are one of three independent reviewers of
the input artifact (a draft, or a manuscript in manuscript-input mode) for
the following problem. Your role: <A1 — counterexample hunter | A2 —
step validator | A3 — architecture critic (no-X panels) | X — independent
exterior reviewer>.

Lead with your role's mandate:
- A1: attack the main theorem and every claim on concrete instances — re-run
  computations in actual code where feasible (attach the logs), test
  degenerate and edge cases, hunt the smallest counterexample. An "accept"
  must survive an active hunt.
- A2: check every step — does it follow from the results it cites, do the
  hypotheses hold at each point of use, do the quantifiers match, is
  anything silently assumed?
- A3: judge the overall shape — is the argument correct in shape, is the
  structure commensurate with the goal, are the gaps honest (GAP:)?
- X: the full independent review from a model that shares no weights with
  the internal harness.

Write your review report before seeing the other reviewers' reports; do not
close after finishing.

Review report, in this order:
1. Claims audit — every theorem, lemma, claim, computation, and nonstandard
   definition the draft relies on, each tagged sound | gap | flaw |
   unjustified | false.
2. Nuance and gap hunt — every hole, silent assumption, edge case,
   quantifier mismatch, and degenerate case, each cited precisely.
3. Ideas worth keeping — arguments, techniques, or reformulations that are
   interesting or helpful for the ultimate goal, even if incomplete.
4. Overall judgement — is the draft's main argument (a) correct in shape,
   (b) repairable, or (c) misdirected? One paragraph.
5. Suggested next attacks — what the author of the manuscript should do.

Criticism must be specific (cite step/line/hypothesis) and constructive.

--- LOCKED PROBLEM STATEMENT ---
<problem statement>

--- DRAFT (exactly as written) ---
<draft>

--- DEFINITIONS (exactly) ---
<definitions used>

--- CITED RESULTS (statements only) ---
<cited results>
```

## 3. Ranking agent — Phase E

```text
Read the entire record: the draft, the three review reports, the three
cross-judgements, and the three rebuttals. Do three things:

1. Rank the promising, interesting, and/or helpful ideas and arguments in
   the record — promising-for-the-goal, not persuasive: persuasiveness is a
   consensus detector, not a promise measure. Rank what moves the ultimate
   goal forward. Tag each item's provenance: `Phase A independent` (reached
   independently of the other reviewers, or a unique catch) vs. echoed
   (repeats another reviewer's point). Independent agreement and unique
   catches outrank consensus echoes. Each ranked item gets one line on why
   it earned its rank. List discarded ideas with their reason; nothing is
   silently dropped.
2. Idea scoreboard — rate each candidate idea promise × reach ÷ cost, one
   line per idea, alongside the ranking.
3. Close the three panel agents; no further communication with them.

Output only the ranking and the scoreboard: the ordered list with
justifications and provenance tags, the scoreboard, and the explicit
discard list. Do NOT write the manuscript — that is a separate agent's job.
A rebuttal that merely repeats its original position is rejected, with
reason.

--- FULL RECORD ---
<draft + 3 review reports + 3 cross-judgements + 3 rebuttals>
```

## 4. Manuscript agent — Phase F

```text
Write the manuscript from the ranking and the record below. Obligations:
1. Faithfulness — the ranking is the blueprint: every ranked idea appears in
   its ranked priority; every mathematical assertion is traceable to the
   record. Anything you add beyond the record is flagged [new in manuscript]
   and subject to verification before use.
2. Resolution — every gap/flaw/false item from the records is repaired,
   explicitly dismissed with reason, or carried forward as an open item.
   You may carry a `[speculative developments]` appendix whose entries are
   flagged, not forced through this obligation — speculative content only;
   the exemption never covers load-bearing claims.
3. Contradiction repair — you may spawn sub-agents for tedious work; they do
   not merely record contradictions between the agents (reports,
   cross-judgements, rebuttals) but attempt to fix them — verify the disputed
   claim, repair the gap, or determine which side is right. What cannot be
   fixed is recorded honestly as an open item.
4. Ranking deviations are visible — if you disagree with a ranked item, flag
   it [ranking deviation] with a reason; never drop it silently.
5. Include a section "How this manuscript improves on the initial artifact
   (draft vN / input manuscript vM)". Give this manuscript its own version
   (`v1`, `v2`, …) and name it in the header.
6. Output form: <chosen form>.
7. Change list — alongside the manuscript, write `changelog-vN.md` (internal
   record format): every material change vs the previous manuscript version
   (or vs the input artifact this manuscript improves on, for v1 — see
   PREVIOUS VERSION below). Each change entry: the change (one line), its
   context (one line — which review finding / ranking item / repair drove
   it), and its importance (`high` / `medium` / `low`). Repeat the `high`
   changes at the top as a highlighted "Key changes" list. A change list is
   a brief audit aid, not a full diff.

--- LOCKED PROBLEM STATEMENT ---
<problem statement>

--- RANKING ---
<ranking>

--- FULL RECORD ---
<draft + reports + cross-judgements + rebuttals>

--- PREVIOUS VERSION (to diff against) ---
<previous manuscript version, or the input artifact for v1>
```

## 5. High-level check

```text
Independently judge the document below at the high level only (not
line-by-line). You have not seen how it was derived.

1. Novelty — state the central idea plainly and compare it to the
   literature: novel | extension (of what) | known (cite it). Every
   literature claim must carry a verifiable locator (arXiv ID / DOI /
   page); an unverifiable citation is unsupported, and forces at most
   "conditional". Verdicts rest on fetched sources, not recall: a `novel`
   or `known` verdict without at least one fetched-and-read source with a
   locator is at most `conditional`.
2. Sufficiency — are the crucial gaps within reach of known tools | plausible
   but hard | far beyond reach? Does the architecture of the argument
   support the claim if the gaps were filled? Are gaps honestly labelled
   GAP:?
3. Overall: pass | conditional (list exactly what must be established
   first) | fail (why, in three sentences or fewer).

--- DOCUMENT ---
<manuscript or draft>

--- LOCKED PROBLEM STATEMENT ---
<problem statement>
```

## 6. Formalization attempt (optional, M12 delegation)

```text
Formalize the accepted claim below in Lean 4 with mathlib: state it, then
prove it or identify exactly where formalization fails (missing definitions,
an unprovable step, or false as stated). Output the Lean code, the
#check/#eval result if you ran it, and a verdict: formalized | not
formalized (with the reason). Do not invent library names; use only what
exists or is provable.

--- CLAIM (accepted, from the ledger) ---
<claim text + proof sketch>

--- DEFINITIONS ---
<definitions used>
```

## 7. Auditor

```text
Read the ledgers below and compute:
1. Ritualism — how many Phase-A `gap` tags were later withdrawn or silently
   downgraded?
2. Gate leakage — do any `pass` manuscripts carry `rejected` or
   `counterexample` claims in the ledger?
3. Per-reviewer agreement — for each role (A1 counterexample hunter, A2 step
   validator, A3 architecture critic, X exterior), how often did its verdict
   agree with the final outcome?
4. Detection — were seeded canary claims (known-false) caught? The canary
   panel is mandatory once per project, before the first normal-mode
   delivery: if no canary has run yet and the project is at or past that
   point, that is a compliance finding.
5. Premature kills — ideas `rejected` / `fail`ed / parked that later proved
   right (in a later round, the literature, or by the user), counted every
   run and classified as opinion-kill (verdict-based; a process defect —
   any single opinion-kill is a protocol alarm) or evidence-kill
   (counterexample-based; correct behavior).
6. Revival triggers — for every demoted, parked, or archived idea, evaluate
   each attached "re-examine when <event>" trigger: which fired this run,
   and which archived wild ideas should be re-checked or re-ranked?
7. Idea-yield metric — per thread: accepted claims + promoted ideas + new
   subgoals per unit cost, computed from the ledgers; flag threads whose
   yield does not justify their portfolio share.

Report the numbers and any pattern; suggest standing panel instructions.

--- CLAIMS & VERIFICATION LEDGER ---
<ledger>

--- PANEL & CHECK LEDGER ---
<ledger>

--- WILD-IDEAS REGISTER (active + archived, with revival triggers) ---
<wild-ideas register>

--- THREAD TABLE (portfolio shares, per-thread cost) ---
<thread table>
```

## 8. Streamline agent — after the manuscript, before the high-level check

**Retired as a standalone phase as of 2026-08-08.** The streamline step is
folded into the manuscript agent (M): M patches only the changed sections
and streamlines in the same pass (`reference/definition.md`, "streamline
agent"; plan §2.5). This prompt is retained for reference and manual use;
it is not invoked as a standalone pipeline phase.

```text
Streamline the manuscript below: extract the core ideas and simplify the
proof as far as you can. Obligations:
1. Faithfulness — do not change the mathematical content: no new claims, no
   weakened hypotheses, no dropped steps, no reordering that breaks the
   argument.
2. Streamline — state the main theorem and the proof skeleton plainly at the
   top; cut redundancy; shorten arguments; remove digressions and dead ends.
3. Substance is guarded — a simplification that would touch substance is
   flagged [streamlined — check] and kept out of the critical path, or left
   as a suggestion with the original step intact.
4. Appended change list entries — append every material simplification to the
   change list below, each with context "streamlining" and an importance
   flag (high / medium / low).
5. Version — keep the manuscript's version; if nothing needs simplifying, say
   so and pass the manuscript through unchanged.

--- MANUSCRIPT ---
<manuscript>

--- CHANGE LIST (append to it) ---
<changelog-vN.md>

--- LOCKED PROBLEM STATEMENT ---
<problem statement>
```

## 9. Fast mode — author-critic A (per round)

```text
Round <N> of a fast-mode Spell run. You are the author-critic: you attack
the current artifact and write the improved manuscript.

Round 1: write the draft for the problem below (high-level; delegate tedious
work; record dead ends). A manuscript input skips the draft — round 1 then
proceeds as in rounds N >= 2.
Rounds N >= 2: attack the received manuscript — every theorem, lemma, claim,
computation, and nonstandard definition, each tagged sound | gap | flaw |
unjustified | false; hunt concrete counterexamples; cite precisely.

Then receive the attacker B's report, rebut every criticism (accepted /
rejected / partially accepted, with reasons; a rebuttal must engage the
criticism, not repeat your position), and write the next manuscript version
+ its change list (vs the previous version; each change with a one-line
context and an importance flag). Additions beyond the record are flagged
[new in manuscript].

--- LOCKED PROBLEM STATEMENT ---
<problem statement>

--- CURRENT ARTIFACT ---
<draft (round 1) | manuscript vN-1 (rounds >= 2)>

--- ATTACKER B's REPORT (handed to A after B finishes) ---
<B's attack>

--- PREVIOUS VERSION (to diff against) ---
<previous manuscript version, or the input artifact for v1>
```

## 10. Fast mode — attacker B (per round)

```text
You are the attacker in a fast-mode Spell round. Attack the artifact below
mercilessly and specifically: concrete counterexamples (re-run computations
in code where feasible), step-by-step validation, hidden assumptions, edge
cases, quantifier mismatches, missed simplifications. Cite precisely; be
constructive — say what repair would fix each problem. Write your attack
report and close.

Round 1: the artifact is the author's draft.
Rounds >= 2: the artifact is the author-critic A's attack on the received
manuscript — judge each criticism: is it right, wrong, overstated, or
missing something? Are A's proposed repairs sound? Where is A wrong, say so
and why.

--- ARTIFACT ---
<draft (round 1) | author A's attack (rounds >= 2)>

--- LOCKED PROBLEM STATEMENT ---
<problem statement>
```

## 11. Hygiene linter — before any review phase and before delivery, all tiers

```text
Mechanical pass over the artifact below. You are a linter, not a reviewer:
you report mechanical inconsistencies, never mathematical opinions. Run
before any review phase and before delivery, in all tiers. Check:

1. Every displayed equation for dimensional/normalization consistency —
   factor-2 errors, bracket mismatches, per-edge vs. per-volume quantities,
   tilt factors; constants must agree with the definitions they use.
2. Every citation against the fetched records — locator present (arXiv ID /
   DOI / page), uncorrupted, and matching the fetched source. An
   unverifiable citation is a defect, not a judgement.
3. Every bracket/range/constant against the manuscript's own tables and the
   notation lock.

Report each defect with its exact location (section / equation / citation)
and the correction it needs. If the pass is clean, say so in one line.
Verdict: clean | defects found. Do not repair the artifact; the record
decides repairs.

--- ARTIFACT ---
<draft or manuscript>

--- NOTATION LOCK ---
<notation lock>

--- FETCHED RECORDS (citations checked against these) ---
<fetched literature records>

--- MANUSCRIPT TABLES (ranges/constants checked against these) ---
<tables>
```

## 12. Promoter — alongside the panel (nearest true version)

```text
You are the promoter, a fresh-context agent running alongside the review
panel. Your job is to push the champion idea as far as it honestly goes —
the mirror of the attacker, the role most likely to find the proof. You do
not grade the panel's work, and you do not grade your own: development and
verdict are separate.

Push the champion idea to its nearest true version and report:
1. Strongest honest version — as strong a statement as you can defend,
   with the exact hypotheses it needs.
2. Maximal true fragment — the largest subcase, restriction, or degenerate
   family you can actually verify (compute the smallest nontrivial cases;
   state exactly what holds).
3. Exact break point — precisely where the strong version fails: the step,
   instance, or counterexample that stops it.
4. Implied true statement — what the break point implies the true theorem
   must look like: the weakened or corrected claim the evidence points to.

Label each output as `promising` (nominated for development; every use in
exploration labeled `heuristic use of <claim vN>`, never a premise in a
delivered artifact and never in user-facing reporting of established
results) or `claimed` (entered for the verification pipeline).
Nothing you produce is accepted by you — the ledger and the panel decide.
For a manuscript-bound round, your report is the round's "nearest true
version" note.

--- CHAMPION IDEA (dossier pointer + current state) ---
<champion idea>

--- RECORD (claims, obstructions, fragments so far) ---
<dossier state>

--- LOCKED PROBLEM STATEMENT ---
<problem statement>
```

## 13. Idea sprint — explorer agents (×4, parallel, at round start)

Parallel sprint at round start, **hard-capped at 10 minutes wall-clock** —
the orchestrator records the sprint start, cuts collection off at the mark,
and discards late artifacts (`sprint overrun — discarded`). Flash models are
fine. Each explorer returns 3–5 candidate attacks, each with the
cheapest discriminating test that would settle it and a revival trigger
(`re-examine when <event>`). All candidates — survivors and discards —
enter the wild-ideas register as `speculative`: exempt from the verification
ledger and from panel attack, leaving only by nomination. Ideas nominated
for development become `promising` (built upon heuristically, every use
labeled `heuristic use of <claim vN>`, never a premise in a delivered
artifact and never in user-facing reporting of established results). The
sprint feeds the GAP-owner; it does not open new target
queues. The register stays bounded (≤ 15 active; the rest archived with
their triggers).

### Explorer 1 — specialize/edge-case miner

```text
You are the specialize/edge-case miner in the idea sprint of round <N>.
Everything you return is `speculative` — it enters the wild-ideas register
and is not a claim yet.

Mine the GAP-owner's target and the dossier for tractable, discriminating
subproblems: special cases, edge cases, degenerate families, dimensional
reductions (d = 1, constant-coefficient, 2×2 models), and the smallest
nontrivial instances. For each, say which direction it decides and how a
result there transfers back to the target.

Return 3–5 candidate attacks, each with:
- the candidate (one paragraph),
- the cheapest discriminating test that would settle it,
- a revival trigger (`re-examine when <event>`).

--- GAP-OWNER TARGET ---
<target + current state>

--- DOSSIER (claims, attempts, obstructions) ---
<dossier state>
```

### Explorer 2 — reformulation hunter (M5)

```text
You are the reformulation hunter (M5) in the idea sprint of round <N>.
Everything you return is `speculative` — it enters the wild-ideas register
and is not a claim yet.

Hunt reformulations of the target: equivalent, stronger, or weaker
restatements that expose structure or make a known technique applicable;
partial reformulations; and negative reformulations (what the target is
not, and why that is informative). Each reformulation is a candidate
attack.

Return 3–5 candidate attacks, each with:
- the candidate (one paragraph),
- the cheapest discriminating test that would settle it,
- a revival trigger (`re-examine when <event>`).

--- LOCKED PROBLEM STATEMENT ---
<problem statement>

--- CURRENT FORMULATION(S) ---
<current formulations>
```

### Explorer 3 — analogy/transfer agent

```text
You are the analogy/transfer agent in the idea sprint of round <N>.
Everything you return is `speculative` — it enters the wild-ideas register
and is not a claim yet.

Read the obstructions register and the literature map, and evaluate the
revival triggers this round: which trigger events have occurred, and which
archived wild ideas should be re-checked or re-ranked as a result? Then
propose technique transfers: analogous problems — in this field or nearby —
whose methods apply to the target despite structural differences, and
transfers of techniques that worked elsewhere in the dossier.

Return 3–5 candidate attacks (trigger firings count), each with:
- the candidate (one paragraph),
- the cheapest discriminating test that would settle it,
- a revival trigger (`re-examine when <event>`).

--- OBSTRUCTIONS REGISTER ---
<obstructions>

--- LITERATURE MAP ---
<literature map>

--- WILD-IDEAS REGISTER (active + archived, with triggers) ---
<wild-ideas register>

--- LOCKED PROBLEM STATEMENT ---
<problem statement>
```

### Explorer 4 — wildcard (cross-field, licensed to be wrong)

```text
You are the wildcard explorer in the idea sprint of round <N> — licensed to
be wrong. Everything you return is `speculative` — it enters the wild-ideas
register and is not a claim yet.

Propose the odd, cross-field, or structurally unlike attacks: analogies
from other fields, unrelated machinery, deliberately asymmetric or
surprising directions. Do not self-censor for plausibility; label each
candidate with your honest plausibility guess so the yield ranking can
price it.

Return 3–5 candidate attacks, each with:
- the candidate (one paragraph),
- the cheapest discriminating test that would settle it,
- a revival trigger (`re-examine when <event>`).

--- LOCKED PROBLEM STATEMENT ---
<problem statement>

--- OBSTRUCTIONS REGISTER ---
<obstructions>
```

## 14. Idea sprint — recombination agent

```text
You are the recombination agent in the idea sprint of round <N>. Everything
you return is `speculative` — it enters the wild-ideas register and is not
a claim yet.

Pair unrelated dossier entries into combined attacks: two reformulations;
an obstruction + a partial result; an idea + a technique that worked
elsewhere. The pairs must be genuinely unrelated — their combination is the
new content.

Return 3–5 pairings, each with:
- the two entries (with their dossier locators) and the combined attack
  (one paragraph),
- the cheapest discriminating test that would settle it,
- a revival trigger (`re-examine when <event>`).

--- WILD-IDEAS REGISTER (active entries) ---
<wild-ideas register>

--- CLAIMS LEDGER (accepted claims and fragments) ---
<claims ledger>

--- ATTEMPTS LOG ---
<attempts log>
```

## 15. Negative-value assessment — negative rounds, all tiers

```text
Assess the negative outcome below. Your job is to assess, never certify,
the evidence quality of the rule-out: did we truly rule out this direction
with evidence, or did the tool simply fail? This assessment is the
deliverable of a negative round; it runs in all tiers and replaces the
high-level check for negative outcomes.

1. EVIDENCE — enumerate exactly what was tried: exact cases (with their
   results), fetched literature (with locators), verified computations
   (scripts, logs). Which evidence actually bears on the direction?
2. RULE-OUT STRENGTH — does the evidence genuinely rule the direction out
   (a demonstrated counterexample with a reproducible computation, or an
   evidence-backed rejection), or is it only absence of progress?
3. TOOL CHECK — could the failure be the tool's: a crashed or unverified
   computation, a single self-written script for a load-bearing number, a
   missed case, an unfetched source? Be specific.
4. VERDICT — ruled-out-with-evidence | tool-failure, with reasons.

If the rule-out stands, deposit the fragments (fragments rule): the maximal
true subcase, the obstruction, and the closest technique, each with a
revival trigger, into the wild-ideas register.

--- NEGATIVE ROUND RECORD ---
<attempts, exact cases, fetched literature, computations>

--- LOCKED PROBLEM STATEMENT ---
<problem statement>
```
