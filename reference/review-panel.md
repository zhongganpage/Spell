# Spell — The Review Panel

The review panel converts a **draft** — or, in manuscript-input mode, a
**manuscript** — into the next **manuscript** (definitions in
`definition.md`). It is adversarial by design: every claim in the input is
attacked, and every attack is itself attacked and rebutted.

## Purpose

- Find every gap, error, hidden assumption, and missed opportunity in the
  draft.
- Surface ideas in the draft that are interesting or helpful for the ultimate
  goal, even if they are not yet correct.
- Produce a manuscript that is strictly better than the draft: every criticism
  has been weighed, every rebuttal heard, and every surviving idea ranked.

## Composition and independence

The panel consists of **five agents**:

- **Three panel reviewers** — **A1** (counterexample hunter), **A2** (step
  validator), and the **third reviewer** — either the **exterior agent X**
  (an independent reviewer from a different provider, configured at project
  start) or, when the user has no X, an internal **A3** (architecture
  critic). A1 and A2 always carry their explicit jobs; X stays an
  independent reviewer; when X is unavailable the panel runs A1, A2, A3,
  each with an explicit job.
- **One ranking agent** (R) — weighs the record, ranks the ideas, and closes
  the panel (Phase E).
- **One manuscript agent** (M) — writes the manuscript from the ranking
  (Phase F). M is a different agent from R.
- **The promoter** — a fresh-context agent alongside the panel, not one of
  the five panel agents; described under "The promoter" below.

Independence rules (non-negotiable):

1. Each agent has a **fresh context**: it has not seen the author's session,
   the author's reasoning, its false starts, or its confidence.
2. Each agent receives **only written records**: the draft, the definitions it
   uses, and the statements of the cited results. Never the derivation story.
3. No agent is told the expected outcome — "this should be right" is
   forbidden.
4. **Model diversity is structural.** A1 and A2 run on the internal harness;
   X runs on the external provider and shares none of its weights, priors,
   training data, or history. Where a different backend is available for the
   internal agents, use it; at minimum, a fresh context window.
5. The three panel reviewers communicate only through written records, in the
   exact sequence of the phases below. R and M each enter once, read the
   record, write their artifact, and close. The promoter reads the same
   records and writes its note without any interaction with the reviewers.
6. **Role diversity is enforced.** A1 is the counterexample hunter, A2 the
   step validator, and the third slot is X (independent exterior reviewer) or
   A3 (architecture critic) when there is no X. When X is unavailable, the
   run records `X unavailable — reduced diversity`; when `X_MODEL` is the
   same provider family as the internal harness, it records `reduced
   diversity`. Either way the panel row carries a **confidence downgrade**.
7. **Reviewers read verbatim inputs.** Definitions and cited statements are
   copied from the dossier's locked sections; any restatement by the author
   is flagged `[restated]` and may be rejected by a reviewer.
8. **The author never fills the reviewers' prompts.** The harness assembles
   each reviewer's input; the author being reviewed has no hand in what the
   reviewers see.

Each panel agent adopts the working stance: *I am the sharpest reviewer in
this room; no nuance, gap, or promising idea will escape me.* Criticism must
be specific (cite the step, the line, the hypothesis) and constructive (say
what repair would fix it). "This is wrong" without a reason is not a review.

**All panel agents run in the background — explicitly.** The harness
launches A1, A2, and X/A3 with its agent tool's explicit background/async
spawn — in an `Agent`-tool harness (Kimi Code, Claude Code),
`run_in_background=true` — and in parallel; R and M are each launched the
same way in their turn. Each agent's written artifact is collected when it
is ready, and a phase starts only when its written inputs exist. Each agent
is spawned with an explicit artifact path and confirms its write in its
final message; an agent that cannot write (read-only type or write-blocked
sandbox) delivers the artifact text in its final message and the
orchestrator persists it verbatim (`protocol.md` §9).

### The exterior agent (X)

X is selected by three configuration variables, fixed once at project start —
the very first startup question (`protocol.md` §10):

| Variable | Meaning | Examples |
|---|---|---|
| `X_PROVIDER` | provider company | `kimi` · `openai` · `anthropic` · `google` · … |
| `X_MODEL` | model name | `k2.7` · `o3` · `claude-sonnet-4` · `gemini-2.5-pro` · … |
| `X_ACCESS` | how X is reached | `api` (HTTP API) · `codex` (local Codex CLI) |

- **Configuration.** Chosen once, at project start: the user supplies the
  three variables when asked during startup (`protocol.md` §10). Current
  default (decided 2026-08-07): `X_PROVIDER=kimi`, `X_MODEL=k2.7`,
  `X_ACCESS=api` (key in `MOONSHOT_API_KEY`). **Codex is pre-installed on
  this machine** (`codex exec`), so `X_PROVIDER=openai`, `X_ACCESS=codex`
  needs no API key.
- **Access modes.**
  - `api` — an HTTP call to the provider's API; the key lives in the
    provider's environment variable (table below) or a secrets store.
  - `codex` — the locally installed OpenAI Codex CLI: X is invoked as
    `codex exec "<phase prompt>"` and its reply is captured as the written
    artifact. It authenticates through Codex's own login (`codex login`) or
    `OPENAI_API_KEY`; nothing extra is stored in the dossier, and `X_MODEL`
    is whatever Codex is configured to use. **Codex may run in a read-only
    sandbox** (`sandbox_mode: read-only`, approval `never`) that blocks file
    writes — then the reply on stdout IS the artifact: capture stdout and
    persist it at the assigned path, marking the record
    `recovered from agent output`.
- **Credentials.** API keys live in environment variables or a secrets store.
  They are never written into the dossier, a report, or any record. If
  `X_ACCESS=api` and the provider's variable is unset, X is unavailable.
- **Common providers.** The usual choices; env var names follow each
  provider's official docs (the ones shown are the common conventions).
  "Aggregators" expose many models behind one key.

  | Provider | Company | Model family | Access (typical env var / command) |
  |---|---|---|---|
  | OpenAI | OpenAI | GPT · o-series · Codex | `OPENAI_API_KEY` or `codex exec` (installed) |
  | Anthropic | Anthropic | Claude (Sonnet · Opus) | `ANTHROPIC_API_KEY` |
  | Google | Google DeepMind | Gemini (Pro · Flash) | `GEMINI_API_KEY` |
  | Kimi | Moonshot AI | k2.x | `MOONSHOT_API_KEY` |
  | DeepSeek | DeepSeek | deepseek-chat · deepseek-reasoner | `DEEPSEEK_API_KEY` |
  | Qwen | Alibaba Cloud | qwen-max · qwen3 | `DASHSCOPE_API_KEY` |
  | Doubao | ByteDance | Doubao | `ARK_API_KEY` |
  | Hunyuan | Tencent | Hunyuan | per Tencent docs |
  | ERNIE | Baidu | ERNIE | `QIANFAN_*` (per docs) |
  | GLM | Zhipu AI | GLM-4.x | `ZHIPUAI_API_KEY` |
  | Grok | xAI | Grok | `XAI_API_KEY` |
  | Mistral | Mistral AI | Mistral Large · Medium | `MISTRAL_API_KEY` |
  | Llama | Meta | Llama 3/4 | via aggregators (`GROQ_API_KEY` · `TOGETHER_API_KEY`) |
  | Aggregators | OpenRouter · Groq · Together | many models | `OPENROUTER_API_KEY` · `GROQ_API_KEY` · `TOGETHER_API_KEY` |

- **Participation.** X is a full panel reviewer: it writes a Phase A review
  report, receives the other two reports, writes a Phase C cross-judgement,
  and writes a Phase D rebuttal — in the same formats as A1 and A2.
- **Availability.** If X cannot be reached when a panel run starts, the panel
  may proceed with A1 and A2 alone. The run records `X unavailable — reduced
  diversity` in the panel ledger, and the manuscript carries the same mark
  (`protocol.md` §8). If X fails mid-run, the written artifacts it produced
  up to the failure point stand, and the ledger records the partial
  participation.
- **Diversity enforcement.** At startup the harness verifies that `X_MODEL`
  is not the same provider family as the internal harness. If it is — or X
  is unavailable — the run is auto-labeled `reduced diversity` with a
  confidence downgrade. One-line check, non-negotiable; the auto-label and
  the downgrade are recorded in the panel ledger and carried on the
  manuscript.
- **Orchestration checklist.** A1 + A2 + X are launched in parallel, each
  with the background/async spawn (`run_in_background=true` in an
  `Agent`-tool harness); every artifact's existence is confirmed before the
  next phase starts; and X's
  absence — including "never launched" — is recorded as a ledger event. An
  absent or same-family X is always flagged; there is no silent same-model
  panel.

### The promoter

The **promoter** is a fresh-context agent that runs alongside the panel (in
manuscript-bound rounds) and pushes the champion idea as far as it goes: the
strongest honest version, the maximal true fragment, exactly where it breaks,
and what the break implies the true statement must be. The mirror of the
attacker; the role most likely to find the proof. It reads the same written
records as the panel — the input artifact, the definitions, and the locked
problem statement — and writes its **"nearest true version" note**, which
enters the ledger as `promising`/`claimed`. The promoter's own work never
grades itself (invariant 1), and it is not one of the five panel agents: it
does not vote, rank, or rebut. Every manuscript-bound round includes its
note.

## Inputs and outputs

- **Input:** one artifact of a specific version — normally a draft (e.g.
  draft v2), but in **manuscript-input mode** a manuscript is accepted
  directly (a complete document, including a previous round's output); in
  both cases the definitions and cited results it uses and the project's
  locked problem statement (`Q` vN) are attached.
- **Output:** one manuscript of the next version, its **change list**
  (`changelog-vN.md`), plus the full written record — 3 review reports, 3
  cross-judgements, 3 rebuttals (or, on an agreeing panel, 3 combined
  responses under the adaptive depth rule), 1 ranking, 1 **idea scoreboard**,
  and the promoter's "nearest true version" note — kept in the dossier. All
  panel artifacts carry versions.

## The phases

All phases are sequential and written: every artifact is a document the next
phase can read. Nothing that is not recorded may be transmitted. The
**hygiene linter** (`definition.md`) is a deterministic mechanical pass, not
a reviewer; it runs before any review phase and before delivery, in all
tiers — the input artifact is linted before Phase A, and the manuscript
output is linted before it is delivered.

### Phase A — Independent review of the input artifact

Each panel agent reads the input artifact and writes a **review report**. Each
reviewer leads with its role's mandate before the common report:

- **A1 — counterexample hunter.** Attack the draft's main theorem and every
  claim on concrete instances: re-run the computations (actual code where
  feasible, logs attached as written records), test degenerate and edge
  cases, hunt the smallest counterexample. An "accept" must survive an
  active hunt.
- **A2 — step validator.** Check every step: does it follow from the results
  it cites, do the hypotheses hold at each point of use, do the quantifiers
  match, is anything silently assumed?
- **A3 — architecture critic** (no-X panels). Judge the overall shape: is
  the argument correct in shape, is the structure commensurate with the
  goal, are the gaps honest (`GAP:` labels)?
- **X — independent exterior reviewer.** The full review from a model that
  shares no weights with the internal harness.

Then the common report, in order:

1. **Claims audit** — every theorem, lemma, claim, computation, and
   nonstandard definition the draft relies on, each tagged
   `sound` | `gap` | `flaw` | `unjustified` | `false`.
2. **Nuance and gap hunt** — every hole, silent assumption, edge case,
   quantifier mismatch, and degenerate case, each cited precisely.
3. **Ideas worth keeping** — arguments, techniques, or reformulations that
   are interesting or helpful for the ultimate goal, even if incomplete.
4. **Overall judgement** — is the draft's main argument (a) correct in shape,
   (b) repairable, or (c) misdirected? In one paragraph.
5. **Suggested next attacks** — what the author of the manuscript should do.

The three agents write these reports independently and **do not close**:
after finishing, each waits for the other two reports.

### Phase B — Exchange

Each agent receives the other two review reports. This phase exists to make
the hand-off explicit and to prevent premature closing; nothing else happens
in it.

**Adaptive phase depth.** When the three Phase-A verdicts are within one
category (all accept / all repairable / all misdirected), the full
B exchange → C cross-review → D rebuttal sequence collapses into a **single
combined response exchange**: each reviewer answers the other two once, in
one written response, and Phases C and D below are skipped. When the verdicts
genuinely conflict, the full B/C/D sequence runs. Records are never merged —
every position change stays verbatim, so the retraction discipline survives
the collapse. An agreeing panel's written record is 3 review reports + 3
combined responses, not a consensus document.

### Phase C — Cross-review

Each agent reviews the other two review reports with the same critical
standard it applied to the draft, and writes a **cross-judgement**:

- Where the other report is right, and why (agree, with the reason).
- Where it is wrong, overstated, or misses something this agent caught.
- What should survive into the manuscript.

### Phase D — Rebuttal

Each agent writes a **rebuttal**: a response to every criticism directed at
its own review report by the other two agents (their Phase A content and
their Phase C cross-judgements). For each criticism: `accepted` (and what
changes), `rejected` (and why), or `partially accepted`. A rebuttal that
merely repeats the original position without engaging the criticism is not
accepted — the ranking agent rejects it in Phase E.

### Phase E — Ranking and closing

A **new agent, the ranking agent R**, enters with fresh context and reads
the entire record: the draft, the three review reports, the three
cross-judgements, and the three rebuttals (or the three combined responses,
on an agreeing panel under the adaptive depth rule).

R does three things:

1. **Ranks the ideas that are promising-for-the-goal** — not "persuasive"
   (persuasiveness is a consensus detector, not a value ranking). Each
   finding carries a **provenance tag**: `Phase A independent` (the reviewer
   reached it on its own) vs. `echoed` (it repeats another report). Unique
   catches and independent agreement outrank consensus echoes. Each ranked
   idea carries one line on why it earned its rank; discarded ideas are
   listed with their reason; nothing is silently dropped.
2. **Emits the idea scoreboard.** Each candidate idea is rated
   promise × reach ÷ cost, alongside the ranking.
3. **Closes the three panel agents.** After they have finished their
   rebuttals (or their combined responses), R terminates them. No further
   communication with the panel.

R's ranking is a written artifact: the ordered list with justifications and
the explicit discard list. R does **not** write the manuscript; that is a
separate agent's job (Phase F).

### Phase F — The manuscript

A **new agent, the manuscript agent M** (separate from R), enters with fresh
context and receives the locked problem statement, the ranking, the idea
scoreboard, the full record (draft, reports, cross-judgements, rebuttals),
and the artifact to diff against — the previous manuscript version, or the
input artifact for v1. M writes the **manuscript**, under these obligations:

1. **Faithfulness.** The ranking is the blueprint: every ranked idea appears
   in its ranked priority, and every mathematical assertion in the manuscript
   is traceable to the record (draft, reports, cross-judgements, rebuttals).
   Anything M adds beyond the record is flagged `[new in manuscript]` and is
   subject to the normal verification ledger (`protocol.md` §8) before it may
   be treated as established.
2. **Resolution of every criticism.** Every `gap`/`flaw`/`false` item from
   the records is either repaired in the manuscript, explicitly dismissed
   with the reason, or carried forward as an open item. Entries in the
   `[speculative developments]` appendix (obligation 11) are the one
   exception: they are flagged, not resolved-by-force — speculative content
   only, and the exemption never covers load-bearing claims.
3. **Surgical by default.** M patches only the sections the record changed
   (diff-scoped) instead of rewriting the full document; ledgers grow as
   appendices rather than being rewritten. Artifact size is monotone
   non-growing per round — a violation is a protocol alarm.
4. **Streamlining, in the same pass.** The streamline step is folded into M
   (§ "The streamline step (folded into M)"): state the main theorem and the
   proof skeleton plainly at the top, cut redundancy and dead ends, reorder
   for clarity — **never changing the mathematical content**. A
   simplification that would touch substance is flagged `[streamlined —
   check]` and kept out of the critical path, or left as a suggestion with
   the original step intact. Every material simplification is appended to
   `changelog-vN.md` with context "streamlining"; if nothing needs
   simplifying, M says so.
5. **Delegation and contradiction repair.** M may spawn sub-agents for
   tedious work (`protocol.md` §5.1). Its sub-agents do not merely record the
   contradictions between the panel agents — the review reports,
   cross-judgements, and rebuttals — they attempt to fix them: verify the
   disputed claim, repair the gap, or determine which side is right. What
   cannot be fixed is recorded honestly as an open item with the reason.
6. **Ranking deviations are visible.** If M disagrees with an item in R's
   ranking, it does not silently drop it: the manuscript flags the
   disagreement `[ranking deviation]` with a reason, so the user can audit.
7. **Improvement remarks.** A dedicated section, "How this manuscript
   improves on the initial artifact (draft vN / input manuscript vM)",
   listing the concrete improvements: which gaps were closed, which ideas
   were promoted, which attacks were rebutted.
8. **Version.** The manuscript carries its own version (`v1`, `v2`, …); the
   panel ledger row records the artifact version it was built from and the
   manuscript version it produced.
9. **Form.** The manuscript is written in the project's chosen output form.
10. **Change list.** Alongside the manuscript, M writes `changelog-vN.md` (in
    the internal record format): every material change vs the previous
    manuscript version (or vs the artifact this manuscript improves on, for
    v1), each entry with a one-line context (which review finding / ranking
    item / repair / streamlining drove it) and an importance flag (`high` /
    `medium` / `low`); the `high` changes are repeated at the top as a
    highlighted "Key changes" list. The change list is a brief audit aid, not
    a full diff.
11. **Speculative developments appendix.** M may carry a `[speculative
    developments]` appendix whose entries are flagged `[speculative]` —
    promising but unestablished material from the record — and are exempt
    from obligation 2's "every gap resolved" requirement. Speculative content
    only; the exemption never covers load-bearing claims.

## After the panel

The manuscript and its change list (`changelog-vN.md`) are handed to the
**high-level check** (`high-level-check.md`) in normal mode; fast rounds skip
the check and deliver. In all tiers the **hygiene linter** runs on the
manuscript before delivery. The panel ledger row records both versions. It
is not yet delivered, and it is not yet "established": its claims enter the
verification ledger like any other claim.

Before the panel closes, its **insights are routed back into the loop**:
Phase-A "suggested next attacks" and R's ranking are appended verbatim to the
dossier's Open Threads, so the best research advice the system generates
informs the next working-loop session, not only the manuscript agent.

## The streamline step (folded into M)

There is **no standalone streamline agent (S)**. As of 2026-08-08 the
streamline step is folded into the manuscript agent (M) and runs inside
Phase F, in the same pass as the surgical patch (obligation 4): the
manuscript writer streamlines the manuscript it is writing. There is no S
phase and no S row in the panel ledger.

M streamlines while writing:

1. **Extract the core ideas.** State the main theorem and the proof skeleton
   plainly at the top; make the architecture of the argument visible.
2. **Simplify the proof.** Cut redundancy, shorten arguments, remove
   digressions and dead ends, reorder for clarity.
3. **Never change the mathematical content.** No new claims, no weakened
   hypotheses, no dropped steps, no reordering that breaks the argument. A
   simplification that would touch substance is flagged `[streamlined —
   check]` and kept out of the critical path, or left as a suggestion with
   the original step intact.
4. **Append to the change list.** Every material simplification is appended
   to `changelog-vN.md` with context "streamlining" and an importance flag.
5. **Version.** The manuscript keeps the version M wrote for it; the
   streamlining is recorded as part of M's panel ledger row. If nothing needs
   simplifying, M says so in its final message and passes the manuscript
   through unchanged.

The streamline step is a conservative edit folded into the manuscript pass:
it improves presentation and focus, never the claim.

## Fast mode

At the start of each round the user chooses **normal mode** (the 5-agent
panel, phases A–F) or **fast mode** — a 2-agent adversarial loop that
replaces the panel for that round. **In a fast round the 5-agent panel does
not run at all:** A1, A2, X/A3, R, M, and the promoter are not spawned and
phases A–F do not occur; the round's panel work is exactly the two-agent loop
below:

- **Round 1.** Agent **A** writes the draft (high-level; tedious work
  delegated per `protocol.md` §5.1). A **manuscript input** skips the draft —
  round 1 then proceeds as rounds ≥ 2. Agent **B** attacks the draft in the
  background while A waits; when B's attack arrives, A rebuts every
  criticism (accepted / rejected / partially accepted, with reasons) and
  writes the manuscript + its change list.
- **Rounds ≥ 2.** The input is the previous manuscript. Agent **A** attacks
  it — every claim tagged, counterexamples hunted, each criticism cited
  precisely. Agent **B** attacks A's attack — is each criticism right, wrong,
  overstated, or missing something? Are A's proposed repairs sound? Then **A
  rebuts** B's critique of its attack and writes the next manuscript + change
  list.

The round then continues with the **hygiene linter**, which runs on the
manuscript before delivery in all tiers; **the high-level check is skipped in
fast mode** (it is the normal-mode gate) and the round delivers after the
linter. There is no standalone streamline step here either — the streamline
obligations (Phase F, obligation 4) fold into the fast-mode manuscript pass,
so the manuscript writer streamlines in the same pass that produces the
manuscript and its change list. Versioning, the change list, question.md
subgoals, and round timing apply unchanged.

**Why it is marked, not certified.** Fast mode trades review independence for
speed: the author attacks its own artifact, there is no exterior reviewer, no
ranking agent, and no cross-judgement phase. Every fast round is marked
`fast mode` in the panel ledger, and the manuscript, verdict, and delivery
note carry the same mark with a **confidence downgrade** — fast mode is a
faster, weaker loop, not a silent substitute for the panel. (Ready-to-paste
prompts: `modules/prompts.md` §9–10.)

## Failure modes to watch for

- **Ritualistic adversariality** — agents manufacturing objections to appear
  critical. Countered by the rule that every criticism must be specific and
  every rebuttal must engage.
- **Groupthink** — A2 and A3 rubber-stamping A1 without independent checking.
  Countered by Phase C, which forces each agent to judge the others, and by
  the exterior agent X, which does not share the internal models' blind
  spots.
- **Faithless ranking** — R ranking by whim rather than by the record.
  Countered by the requirement that every ranked item carries a justification
  and every discard a reason; the provenance tags (`Phase A independent` vs.
  `echoed`) let the user audit where each ranked finding came from.
- **Faithless writing** — M ignoring the ranking and the record and rewriting
  from scratch. Countered by Phase F obligations 1 and 2; the user can audit
  M against the ranking and the record.
- **Ranker–writer drift** — M silently deviating from R's ranking. Countered
  by the `[ranking deviation]` flag, which makes disagreements visible rather
  than silent.
- **Streamline drift** — M simplifying past the mathematical content.
  Countered by the rule that a simplification touching substance is flagged
  `[streamlined — check]` and kept out of the critical path, or left as a
  suggestion with the original step intact.
- **Silent degradation** — the panel proceeding without X, or with a
  same-family X, and nobody recording it. Countered by the startup diversity
  check (a same-family or absent X auto-labels the run `reduced diversity`
  with a confidence downgrade), the mandatory `X unavailable — reduced
  diversity` ledger entry and the same mark on the manuscript, and the
  orchestration checklist that records any X absence — including "never
  launched" — as a ledger event.
- **Exterior-agent drift** — the provider returning a different model than
  configured. Countered by recording the actual model identifier reported by
  the provider in each run's panel row.
