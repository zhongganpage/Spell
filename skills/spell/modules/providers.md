# Spell — Exterior Agent (X): configuration

Load this module at project start — configuring X is the **very first
startup question** (`protocol.md` §10) — or when wiring X. X gives the panel
genuine model diversity — it is the one reviewer that does not share the
internal harness's weights and blind spots.

## Variables (fixed once at project start)

| Variable | Meaning | Examples |
|---|---|---|
| `X_PROVIDER` | provider company | `kimi` · `openai` · `anthropic` · `google` · … |
| `X_MODEL` | model name | `k2.7` · `o3` · `claude-sonnet-4` · `gemini-2.5-pro` · … |
| `X_ACCESS` | how X is reached | `api` (HTTP API) · `codex` (local Codex CLI) |

Current default (decided 2026-08-07): `X_PROVIDER=kimi`, `X_MODEL=k2.7`,
`X_ACCESS=api` (key in `MOONSHOT_API_KEY`). Codex is pre-installed on this
machine (`codex exec`), so `X_PROVIDER=openai`, `X_ACCESS=codex` needs no
API key.

## Access modes

- **`api`** — HTTP call to the provider's API; the key lives in the
  provider's environment variable (table below) or a secrets store. If the
  variable is unset, X is unavailable.
- **`codex`** — the locally installed OpenAI Codex CLI: invoke X as
  `codex exec "<phase prompt>"` and capture the reply as the written
  artifact. Authenticates through Codex's own login or `OPENAI_API_KEY`;
  `X_MODEL` is whatever Codex is configured to use.

Credentials are never written into the dossier, a report, or any record.

## Common providers

Env var names follow each provider's official docs (the ones shown are the
common conventions). "Aggregators" expose many models behind one key.

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

## Availability

X participates in all phases (A–D) in the same formats as A1 and A2. If X
cannot be reached at panel start, the panel proceeds with A1 and A2 alone;
the run records `X unavailable — reduced diversity` in the panel ledger and
the manuscript carries the same mark. If X fails mid-run, its written
artifacts up to the failure point stand.
