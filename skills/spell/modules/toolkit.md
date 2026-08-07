# Spell — Toolkit (load when stuck)

## Transformation toolkit (M1–M11)

Run these moves *mechanically*; each move, even a failed one, is recorded.

- **M1 — Compute examples.** Smallest cases by hand or code; build a table;
  state the pattern as a conjecture (recorded); then hunt the *smallest
  counterexample* to it.
- **M2 — Extremes and limits.** Degenerate cases: zero, empty set, identity,
  trivial group, $n=0$, $p=\infty$, $\varepsilon \to 0$, $n \to \infty$.
  Extremes expose the mechanism that intermediate cases hide.
- **M3 — Specialize.** Restrict to a subfamily provable with known tools;
  prove it completely — its proof usually reveals the general mechanism.
- **M4 — Generalize.** Strip hypotheses to the minimal setting. The proof
  often falls out of the cleaner statement.
- **M5 — Reformulate.** Translate the problem into a different language:
  fixed-point, optimization, variational, combinatorial, probabilistic,
  graph-theoretic, or generating-function. Change variables, normalize, take
  the dual, complement, contrapositive. Every new form is new search fuel.
- **M6 — Prove partial directions.** The necessary direction; a weaker
  bound; a restricted class. A half-proved statement with an exact boundary
  of where the proof stops is a research result in itself.
- **M7 — Key-lemma hunt.** State the lemma that would solve the problem: "If
  Lemma X, then the theorem follows." Prove X standalone, or find its
  counterexample. The bottleneck is usually the whole problem in miniature.
- **M8 — Extremal principle.** Assume a minimal counterexample / maximal
  element / worst case and derive a contradiction.
- **M9 — Induction and monotonicity.** Is there a natural induction step? A
  monotonicity that makes a limiting argument work?
- **M10 — Computational experiments.** Brute force over small ranges, random
  search for counterexamples, symbolic tests (OEIS, a CAS). Experiments
  inform; they never substitute for a proof.
- **M11 — Literature triangulation.** For every reformulation, search its
  terminology. Find the nearest known theorem and read its proof *fully* —
  adapted, not cited. Check specifically whether the claim (or its negation)
  is already known — this distinguishes "open" from "impossible".

## The stuck ladder

A thread is stalled when two consecutive sessions on it produced no progress.
Escalate in order; you may not skip a rung:

1. **Untried moves.** Run the toolkit moves you have not yet tried (keep a
   tick list per thread).
2. **Re-read the dossier.** There are usually unused threads and
   half-written entries from earlier sessions.
3. **New reformulation → new search.** Each new form of the problem is a new
   set of literature search terms.
4. **Reduce to a baby case.** Prove a trivial case completely and write it up
   as if for publication.
5. **Split the problem.** Prove *any* provable subproblem and record it as a
   partial result — a durable asset.
6. **The floor — prove something anyway.** If no next step exists even now,
   the session must not end empty-handed: produce a **trial proof** (best
   argument you can write, every gap labelled `GAP:` in the text) or a
   **proof for a semi-explicit example** (a concrete instance worked through
   completely). Trivial is fine and encouraged; the point is engagement. A
   trial proof is not a claim of correctness — record it as
   draft/`claimed` and let verification judge it.
7. **Only now:** mark the thread `stalled` — with the reason and a one-line
   "resume here" — and open a new thread. A stalled thread is a storage
   state, not a verdict. The problem itself never becomes abandoned.
