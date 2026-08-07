# Spell — The Review Panel

The review panel converts a **draft** into a **manuscript** (definitions in
`definition.md`). It is adversarial by design: every claim in the draft is
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
   record, write their artifact, and close.
6. **Role diversity is enforced.** A1 is the counterexample hunter, A2 the
   step validator, and the third slot is X (independent exterior reviewer) or
   A3 (architecture critic) when there is no X. When X is unavailable, the
   run records `X unavailable — reduced diversity` and a **confidence
   downgrade** on the panel row.
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
    is whatever Codex is configured to use.
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

## Inputs and outputs

- **Input:** one draft of a specific version (e.g. draft v2, plus the
  definitions and cited results it uses) and the project's locked problem
  statement (`Q` vN).
- **Output:** one manuscript of a specific version, plus the full written
  record — 3 review reports, 3 cross-judgements, 3 rebuttals, 1 ranking —
  kept in the dossier. All panel artifacts carry versions.

## The phases

All phases are sequential and written: every artifact is a document the next
phase can read. Nothing that is not recorded may be transmitted.

### Phase A — Independent review of the draft

Each panel agent reads the draft and writes a **review report**. Each
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
cross-judgements, and the three rebuttals.

R does two things:

1. **Ranks** the persuasive, interesting, and/or helpful ideas and arguments
   in the record — the ones the manuscript must be built on — each with one
   line on why it earned its rank. Discarded ideas are listed with their
   reason; nothing is silently dropped.
2. **Closes the three panel agents.** After they have finished their
   rebuttals, R terminates them. No further communication with the panel.

R's ranking is a written artifact: the ordered list with justifications and
the explicit discard list. R does **not** write the manuscript; that is a
separate agent's job (Phase F).

### Phase F — The manuscript

A **new agent, the manuscript agent M** (separate from R), enters with fresh
context and receives the locked problem statement, the ranking, and the full
record (draft, reports, cross-judgements, rebuttals). M writes the
**manuscript**, under these obligations:

1. **Faithfulness.** The ranking is the blueprint: every ranked idea appears
   in its ranked priority, and every mathematical assertion in the manuscript
   is traceable to the record (draft, reports, cross-judgements, rebuttals).
   Anything M adds beyond the record is flagged `[new in manuscript]` and is
   subject to the normal verification ledger (`protocol.md` §8) before it may
   be treated as established.
2. **Resolution of every criticism.** Every `gap`/`flaw`/`false` item from
   the records is either repaired in the manuscript, explicitly dismissed
   with the reason, or carried forward as an open item.
3. **Delegation and contradiction repair.** M may spawn sub-agents for
   tedious work (`protocol.md` §5.1). Its sub-agents do not merely record the
   contradictions between the panel agents — the review reports,
   cross-judgements, and rebuttals — they attempt to fix them: verify the
   disputed claim, repair the gap, or determine which side is right. What
   cannot be fixed is recorded honestly as an open item with the reason.
4. **Ranking deviations are visible.** If M disagrees with an item in R's
   ranking, it does not silently drop it: the manuscript flags the
   disagreement `[ranking deviation]` with a reason, so the user can audit.
5. **Improvement remarks.** A dedicated section, "How this manuscript
   improves on the initial draft (draft vN)", listing the concrete
   improvements: which gaps were closed, which ideas were promoted, which
   attacks were rebutted.
6. **Version.** The manuscript carries its own version (`v1`, `v2`, …); the
   panel ledger row records the draft version it was built from and the
   manuscript version it produced.
7. **Form.** The manuscript is written in the project's chosen output form.

## After the panel

The manuscript is handed to the **high-level check** (`high-level-check.md`).
It is not yet delivered, and it is not yet "established": its claims enter the
verification ledger like any other claim.

Before the panel closes, its **insights are routed back into the loop**:
Phase-A "suggested next attacks" and R's ranking are appended verbatim to the
dossier's Open Threads, so the best research advice the system generates
informs the next working-loop session, not only the manuscript agent.

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
  and every discard a reason; the user can audit R's ranking against the
  record.
- **Faithless writing** — M ignoring the ranking and the record and rewriting
  from scratch. Countered by Phase F obligations 1 and 2; the user can audit
  M against the ranking and the record.
- **Ranker–writer drift** — M silently deviating from R's ranking. Countered
  by the `[ranking deviation]` flag, which makes disagreements visible rather
  than silent.
- **Silent degradation** — the panel proceeding without X and nobody
  recording it. Countered by the mandatory `X unavailable — reduced
  diversity` ledger entry and the same mark on the manuscript.
- **Exterior-agent drift** — the provider returning a different model than
  configured. Countered by recording the actual model identifier reported by
  the provider in each run's panel row.
