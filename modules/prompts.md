# Spell — Ready-to-Paste Prompts

Load this module when a phase starts. Each prompt is pasted into a fresh
session (internal agents) or passed to X (`codex exec "<prompt>"` /
provider API). Attach the inputs listed under each.

**Delivery contract (the orchestrator applies this to every assembled
prompt).** Add an explicit output path to the prompt, and require the agent
to write its artifact there and confirm the write in its final message. If
the agent's environment is read-only or the write fails (a read-only
sub-agent type, a sandboxed `codex exec`), the agent must instead include
the complete artifact text in its final message; the orchestrator persists
it verbatim at the assigned path and marks the record
`recovered from agent output`. Verify the file exists before the next phase
starts (Invariant rule 9; `protocol.md` §9).

## 1. Claim reviewer — verification ledger

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
cross-judgements, and the three rebuttals. Do two things:

1. Rank the persuasive, interesting, and/or helpful ideas and arguments in
   the record — the ones the manuscript must be built on — each with one
   line on why it earned its rank. List discarded ideas with their reason;
   nothing is silently dropped.
2. Close the three panel agents; no further communication with them.

Output only the ranking: ordered list with justifications, and the explicit
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
Read the two ledgers below and compute:
1. Ritualism — how many Phase-A `gap` tags were later withdrawn or silently
   downgraded?
2. Gate leakage — do any `pass` manuscripts carry `rejected` or
   `counterexample` claims in the ledger?
3. Per-reviewer agreement — for each role (A1 counterexample hunter, A2 step
   validator, A3 architecture critic, X exterior), how often did its verdict
   agree with the final outcome?
4. Detection — were seeded canary claims (known-false) caught?

Report the numbers and any pattern; suggest standing panel instructions.

--- CLAIMS & VERIFICATION LEDGER ---
<ledger>

--- PANEL & CHECK LEDGER ---
<ledger>
```

## 8. Streamline agent — after the manuscript, before the high-level check

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
