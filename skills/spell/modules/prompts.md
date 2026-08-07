# Spell — Ready-to-Paste Prompts

Load this module when a phase starts. Each prompt is pasted into a fresh
session (internal agents) or passed to X (`codex exec "<prompt>"` /
provider API). Attach the inputs listed under each.

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

## 2. Panel reviewer — Phase A (for A1, A2, X)

```text
You are the sharpest reviewer in this room; no nuance, gap, or promising
idea will escape you. You are one of three independent reviewers of a draft
for the following problem. Write your review report before seeing the other
reviewers' reports; do not close after finishing.

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
3. Ranking deviations are visible — if you disagree with a ranked item, flag
   it [ranking deviation] with a reason; never drop it silently.
4. Include a section "How this manuscript improves on the initial draft
   (draft vN)". Give this manuscript its own version (`v1`, `v2`, …) and name
   it in the header.
5. Output form: <chosen form>.

--- LOCKED PROBLEM STATEMENT ---
<problem statement>

--- RANKING ---
<ranking>

--- FULL RECORD ---
<draft + reports + cross-judgements + rebuttals>
```

## 5. High-level check

```text
Independently judge the document below at the high level only (not
line-by-line). You have not seen how it was derived.

1. Novelty — state the central idea plainly and compare it to the
   literature: novel | extension (of what) | known (cite it). Every
   literature claim must carry a verifiable locator (arXiv ID / DOI /
   page); an unverifiable citation is unsupported, and forces at most
   "conditional".
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
