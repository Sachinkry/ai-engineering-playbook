# LLM Model Selection Guide

> A practical, regularly updated framework for choosing the right model — covering capability, cost, context, and operational fit.
>
> **Last updated: March 2026** · [Anthropic](#-anthropic--claude-4x-series) · [OpenAI](#-openai--gpt-5x--reasoning-series) · [Google](#-google--gemini-31--25-series) · [Open Source](#-open-source-models) · [Cost Analysis](#-cost-analysis) · [Ops](#️-operational-considerations)

---

## Contents

- [The Four-Dimension Framework](#the-four-dimension-framework)
- [Anthropic — Claude 4.x](#-anthropic--claude-4x-series)
- [OpenAI — GPT-5.x & Reasoning](#-openai--gpt-5x--reasoning-series)
- [Google — Gemini 3.1 & 2.5](#-google--gemini-31--25-series)
- [Open-Source Models](#-open-source-models)
- [Cost Analysis](#-cost-analysis)
- [Operational Considerations](#️-operational-considerations)
- [At-a-Glance Summary](#-at-a-glance-summary)

---

## The Four-Dimension Framework

Choosing the right LLM is never about picking the most powerful model — it's about matching capabilities to constraints. Work through these four filters before shortlisting anything.

### 1. Capability Tier

| Task Complexity                             | Right Tier   | Examples                                              |
| ------------------------------------------- | ------------ | ----------------------------------------------------- |
| Autonomous agents, complex tool chains, SWE | **Frontier** | Claude Opus 4.6, GPT-5.4, o3                          |
| Text generation, summarisation, Q&A         | **Mid-tier** | Claude Sonnet 4.6, GPT-5.2, Gemini 2.5 Flash          |
| Classification, triage, templated responses | **Budget**   | Claude Haiku 4.5, GPT-5.4-nano, Gemini 2.5 Flash-Lite |

### 2. Context Window

| Range      | Models That Qualify                                                                                   |
| ---------- | ----------------------------------------------------------------------------------------------------- |
| Under 200K | Many budget/mid-tier models (verify per model)                                                        |
| 200K – 1M  | Claude 4.5/Haiku 4.5 (200K), Claude 4.6 (1M); GPT-5.4 (up to 1M); Gemini 2.5/3.1 Pro/Flash (up to 1M) |
| Over 1M    | Llama 4 Scout (10M self-hosted); extended-context variants (verify)                                   |

### 3. Cost Envelope

- **Agentic multiplier**: loops consume 5–15× more tokens than single-turn interactions — model the realistic budget, not the demo budget
- **Prompt caching**: cache reads cost ~10% of standard input price (Anthropic)
- **Batch API**: 50% discount on all Anthropic models for non-real-time workloads
- **Model routing**: route simple sub-tasks to nano/lite tier — typically 60–80% cost reduction

### 4. Operational Constraints

| Constraint              | Approach                                                               |
| ----------------------- | ---------------------------------------------------------------------- |
| Data residency required | Regional endpoints (Anthropic/OpenAI, +10% uplift) or self-hosted OSS  |
| P99 latency < 100ms     | Gemini Flash, o4-mini, or self-hosted Mistral/Qwen3                    |
| Rate limit ceiling      | OpenAI Tier 5: 10K RPM / 10M TPM · Anthropic Tier 4: 4K RPM / 400K TPM |
| Zero data egress        | DeepSeek-V3.2, GLM-4.7, Qwen3 self-hosted on-prem                      |

### Decision Quick-Reference

| Scenario                     | Recommended                            | Why                                                 |
| ---------------------------- | -------------------------------------- | --------------------------------------------------- |
| Autonomous coding agent      | Claude Opus 4.6                        | Best SWE-bench scores; reliable multi-step tool use |
| Enterprise RAG > 500K tokens | Claude Sonnet/Opus 4.6, Gemini 3.1 Pro | Native up to ~1M context (provider-dependent)       |
| Customer support at scale    | Claude Haiku 4.5, GPT-5.4-mini         | Sub-100ms; $0.10–$1/MTok input                      |
| Complex math / logic         | o3, o3-pro                             | Thinking-mode; best AIME/MATH-500 scores            |
| Multimodal (video + audio)   | Gemini 3.1 Pro, GPT-5.4                | Native interleaved multimodal architecture          |
| Zero data egress             | DeepSeek-V3.2 (OSS)                    | Self-hosted; no third-party API calls               |
| General production default   | Claude Sonnet 4.6, GPT-5.2             | Best quality-to-cost for most workloads             |
| Ultra-low cost, high volume  | Gemini 2.5 Flash-Lite                  | $0.10/MTok input — cheapest capable option          |

---

## 🟠 Anthropic — Claude 4.x Series

> **Key change in 4.6:** Full 1M token context window available at standard pricing on Sonnet and Opus — no long-context surcharge.

| Model               | Input / MTok | Output / MTok | Batch Input | Context        | Best For                         |
| ------------------- | ------------ | ------------- | ----------- | -------------- | -------------------------------- |
| `claude-opus-4-6`   | $5.00        | $25.00        | $2.50       | **1M**         | Agentic tasks, complex reasoning |
| `claude-sonnet-4-6` | $3.00        | $15.00        | $1.50       | **1M**         | General production default ⭐    |
| `claude-opus-4-5`   | $5.00        | $25.00        | $2.50       | 200K           | Complex reasoning, coding        |
| `claude-sonnet-4-5` | $3.00        | $15.00        | $1.50       | 200K (1M beta) | General production               |
| `claude-haiku-4-5`  | $1.00        | $5.00         | $0.50       | 200K           | High-volume, low-latency         |
| `claude-haiku-3-5`  | $0.80        | $4.00         | $0.40       | 200K           | Cost-sensitive tasks             |

**Prompt Caching multipliers:**

| Operation   | Multiplier           | TTL           |
| ----------- | -------------------- | ------------- |
| Cache write | 1.25× input price    | 5 minutes     |
| Cache write | 2.0× input price     | 1 hour        |
| Cache read  | **0.1× input price** | Same as write |

> **Fast Mode (Opus 4.6 only):** $30/MTok input · $150/MTok output — 6× premium for significantly faster output. Only justified for real-time agentic loops where latency is the primary bottleneck.

---

## 🟢 OpenAI — GPT-5.x & Reasoning Series

> GPT-5.4 is the current flagship. Long-context surcharge applies beyond 200K tokens on select models. Regional processing adds a **10% uplift** for gpt-5.4, gpt-5.4-mini, gpt-5.4-nano, gpt-5.4-pro.

### Standard Series

| Model               | Input / MTok | Cached Input | Output / MTok | Long Context Input (>200K) |
| ------------------- | ------------ | ------------ | ------------- | -------------------------- |
| `gpt-5.4`           | $2.50        | $0.25        | $15.00        | $5.00                      |
| `gpt-5.4-mini`      | $0.75        | $0.075       | $4.50         | —                          |
| `gpt-5.4-nano`      | $0.20        | $0.02        | $1.25         | —                          |
| `gpt-5.4-pro`       | $30.00       | —            | $180.00       | $60.00                     |
| `gpt-5.2`           | $1.75        | $0.175       | $14.00        | —                          |
| `gpt-5` / `gpt-5.1` | $1.25        | $0.125       | $10.00        | —                          |
| `gpt-5-mini`        | $0.25        | $0.025       | $2.00         | —                          |
| `gpt-5-nano`        | $0.05        | $0.005       | $0.40         | —                          |

### o-Series Reasoning Models

| Model     | Input / MTok | Cached Input | Output / MTok | Use When                         |
| --------- | ------------ | ------------ | ------------- | -------------------------------- |
| `o3`      | $2.00        | $0.50        | $8.00         | Complex debug, logic, math       |
| `o4-mini` | $1.10        | $0.275       | $4.40         | Budget reasoning — best value ⭐ |
| `o3-pro`  | $20.00       | —            | $80.00        | Maximum reasoning depth          |

---

## 🔵 Google — Gemini 3.1 & 2.5 Series

### Gemini 3.1 Series — Latest (Preview)

> Preview models may change before becoming stable and carry more restrictive rate limits.

| Model                           | Input / MTok                         | Output / MTok                   | Context | Key Strength                               |
| ------------------------------- | ------------------------------------ | ------------------------------- | ------- | ------------------------------------------ |
| `gemini-3.1-pro-preview`        | $2.00 (≤200K) / $4.00 (>200K)        | $12.00 (≤200K) / $18.00 (>200K) | **1M**  | Flagship: multimodal, agentic, vibe-coding |
| `gemini-3.1-flash-lite-preview` | $0.25 (text/img/vid) / $0.50 (audio) | $1.50                           | **1M**  | Most cost-efficient; high-volume agentic   |
| `gemini-3-flash-preview`        | $0.50 (text/img/vid) / $1.00 (audio) | $3.00                           | **1M**  | Speed + search grounding                   |

### Gemini 2.5 Series — Stable

| Model                   | Input / MTok                         | Output / MTok                   | Context | Key Strength                       |
| ----------------------- | ------------------------------------ | ------------------------------- | ------- | ---------------------------------- |
| `gemini-2.5-pro`        | $1.25 (≤200K) / $2.50 (>200K)        | $10.00 (≤200K) / $15.00 (>200K) | 1M      | Coding & complex reasoning         |
| `gemini-2.5-flash`      | $0.30 (text/img/vid) / $1.00 (audio) | $2.50                           | 1M      | Hybrid reasoning; thinking budgets |
| `gemini-2.5-flash-lite` | $0.10 (text/img/vid) / $0.30 (audio) | $0.40                           | 1M      | At-scale, cheapest capable ⭐      |

**Search Grounding:**

- Gemini 3.x: $14 / 1K queries (after 5,000 free prompts/month)
- Gemini 2.5: $35 / 1K grounded prompts (after 1,500 RPD free)
- Billed per search call, separate from token costs

---

## 🟣 Open-Source Models

> **Quality gap vs. frontier as of March 2026: ~5 index points.** Top open-weight models are viable production choices for most workloads — not just research.

### S-Tier — Frontier-Competitive

#### GLM-4.7 & GLM-5 — Zhipu AI

- **SWE-bench Verified:** 77.8 · **HumanEval:** 94.2 (best open-source) · **GPQA Diamond:** 86.0
- GLM-5 holds the highest Chatbot Arena rating among open models (1451)
- Context: 200K · Hardware: 4× A100 80GB · License: commercial OK
- **Best for:** Coding agents, scientific Q&A, complex multi-turn reasoning

#### Kimi K2.5 — Moonshot AI

- **AIME 2025:** 96% — outperforms most proprietary models on math
- Built as a thinking agent; strong multi-step tool chain stability
- Context: 256K
- **Best for:** Math-heavy workflows, research automation, deep reasoning agents

#### DeepSeek-V3.2 — DeepSeek

- **LiveCodeBench:** 90% (best open-source coding) · **MMLU:** 94.2%
- Sparse Attention (DSA) for efficient long-context inference; 671B MoE architecture
- Hardware: 8× H100 (full BF16), 4× H100 (Q4) · License: MIT-like (commercial OK)
- **Best for:** General-purpose agentic systems, coding, knowledge-intensive tasks

#### Qwen3-235B & Qwen3.5-397B-A17B — Alibaba

- **MATH-500:** 97.8% (Thinking Mode) — tops math benchmarks among all models
- Qwen3.5 adds native multimodal reasoning with 8.6–19× higher decoding throughput
- License: Apache 2.0
- **Best for:** Math, multilingual, multimodal reasoning, document QA (Qwen3-VL)

### A-Tier — High Quality, Specific Strengths

| Model                       | Params      | Context | Standout                | Best For                   |
| --------------------------- | ----------- | ------- | ----------------------- | -------------------------- |
| Llama 4 Scout (Meta)        | 109B active | **10M** | Extreme context         | Massive document ingestion |
| Llama 4 Maverick (Meta)     | 400B MoE    | 1M      | General multimodal      | Visual-language tasks      |
| MiMo-V2-Flash               | ~50B        | 256K    | 87% LiveCodeBench       | Coding, real-time agents   |
| MiniMax M2.5                | Large MoE   | ~205K   | Human preference leader | Customer-facing chat       |
| Nemotron Super 49B (NVIDIA) | 49B         | 128K    | Efficient agentic       | Scalable GPU agents        |

> **Llama 4 note:** Meta's license restricts commercial use for services exceeding 700M monthly active users. Not a constraint for most teams — but verify at consumer scale.

### Open-Source API Cost (via managed providers)

| Model           | Approx. API Cost ($/MTok input) | Self-Host Hardware | License       |
| --------------- | ------------------------------- | ------------------ | ------------- |
| DeepSeek-V3.2   | $0.20–0.50                      | 4–8× H100          | MIT-like      |
| GLM-4.7 / GLM-5 | $0.30–0.60                      | 4× A100 80GB       | Commercial OK |
| Qwen3-235B      | $0.30–0.80                      | 4–8× A100 (Q4)     | Apache 2.0    |
| Llama 4 Scout   | $0.20–0.40                      | 4× H100 (Q4)       | Meta Llama 4  |
| Kimi K2.5       | $0.30–0.60                      | 4× A100 80GB       | Open weights  |
| MiMo-V2-Flash   | $0.20–0.40                      | 2× A100 (Q4)       | Apache 2.0    |

---

## 💰 Cost Analysis

### Monthly Cost at Scale

> Assumes 1M queries/month · 1K input tokens + 500 output tokens per query · no caching

| Model                 | 10K queries/mo | 100K queries/mo | 1M queries/mo |
| --------------------- | -------------- | --------------- | ------------- |
| Claude Opus 4.6       | $17.50         | $175            | $1,750        |
| Claude Sonnet 4.6     | $10.50         | $105            | $1,050        |
| Claude Haiku 4.5      | $3.50          | $35             | $350          |
| GPT-5.4               | $16.25         | $162.50         | $1,625        |
| GPT-5.4-mini          | $5.00          | $50             | $500          |
| GPT-5.4-nano          | $1.35          | $13.50          | $135          |
| Gemini 3.1 Flash-Lite | $1.25          | $12.50          | $125          |
| Gemini 2.5 Flash-Lite | $0.35          | $3.50           | **$35**       |

> 💡 Gemini 2.5 Flash-Lite is 50× cheaper than Opus 4.6 for simple high-volume workloads.

### Agentic Cost Multiplier

Agentic loops consume **5–15× more tokens** than single-turn interactions:

- Simple 3-step agent (plan → act → verify): ~3–5K tokens per query
- Complex coding agent (10-step repo edit): ~20–80K tokens per task

**Cost reduction levers:**

1. **Prompt caching** on static system prompts / tool schemas → 40–60% savings
2. **Batch API** (Anthropic) for non-real-time workloads → 50% off
3. **Model routing** — classify and route simple steps to nano/lite tier → 60–80% savings
4. **Token budgeting** — set `max_tokens` per step; alert on p95/p99 usage

### Anthropic Batch API Pricing

| Model             | Standard Input | Batch Input | Standard Output | Batch Output |
| ----------------- | -------------- | ----------- | --------------- | ------------ |
| claude-opus-4-6   | $5.00          | $2.50       | $25.00          | $12.50       |
| claude-sonnet-4-6 | $3.00          | $1.50       | $15.00          | $7.50        |
| claude-haiku-4-5  | $1.00          | $0.50       | $5.00           | $2.50        |

---

## ⚙️ Operational Considerations

### Rate Limits

| Provider  | Tier                | Req/min | Tokens/min |
| --------- | ------------------- | ------- | ---------- |
| Anthropic | Tier 1 (basic)      | 50 RPM  | 40K TPM    |
| Anthropic | Tier 4 (enterprise) | 4K RPM  | 400K TPM   |
| OpenAI    | Tier 1 (basic)      | 500 RPM | 30K TPM    |
| OpenAI    | Tier 5 (enterprise) | 10K RPM | 10M TPM    |

### API vs. Self-Hosting

| Factor         | Use Cloud API             | Self-Host               |
| -------------- | ------------------------- | ----------------------- |
| Volume         | Under 1M queries/month    | Over 10M queries/month  |
| Data privacy   | Standard compliance OK    | Zero-trust, air-gapped  |
| Latency        | Acceptable >200ms P99     | Need <100ms P99         |
| Team           | No GPU infra expertise    | Has GPU/MLOps team      |
| Cost crossover | Variable workloads        | Predictable high-volume |
| Model updates  | Automatic                 | Manual, controlled      |
| Fine-tuning    | API fine-tuning (limited) | Full weight access      |

> ⚠️ **Hidden cost of self-hosting:** 1–2 dedicated ML infra engineers, 8–16 week GPU procurement lead times, model version management, monitoring, and on-call burden. Always model TCO, not just token cost.

### Multi-Provider Reliability

Never rely on a single provider in production. Implement **semantic fallback** — jump to a different provider on failure, not just retry the same model:

```text
Primary:    Claude Sonnet 4.6
Fallback 1: GPT-5.2  (different provider — avoids correlated failures)
Fallback 2: Cached / simplified response
Last resort: Self-hosted OSS model
```

---

## 📊 At-a-Glance Summary

| Use Case                    | Top Pick               | Budget Alternative    | Open-Source             |
| --------------------------- | ---------------------- | --------------------- | ----------------------- |
| Autonomous dev agent        | Claude Opus 4.6        | Claude Sonnet 4.6     | GLM-4.7 / DeepSeek-V3.2 |
| Enterprise RAG (large docs) | Claude Sonnet/Opus 4.6 | Gemini 3.1 Flash-Lite | Llama 4 Scout (10M ctx) |
| Customer support at scale   | Claude Haiku 4.5       | GPT-5.4-nano          | MiMo-V2-Flash           |
| Math / logic reasoning      | o3 / o3-pro            | o4-mini               | Kimi K2.5 / Qwen3-235B  |
| Multimodal (video/audio)    | Gemini 3.1 Pro Preview | Gemini 2.5 Flash      | Qwen3.5-VL              |
| Private / zero data egress  | _(use open-source)_    | _(use open-source)_   | DeepSeek-V3.2 / GLM-4.7 |
| General production default  | Claude Sonnet 4.6      | GPT-5.2               | Qwen3-235B              |
| Ultra-low cost, high volume | Gemini 2.5 Flash-Lite  | GPT-5.4-nano          | MiMo-V2-Flash           |

---

## Changelog

| Date       | Changes                                                                                                                                                          |
| ---------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| March 2026 | Added Gemini 3.1 Pro/Flash-Lite preview pricing; updated GPT-5.x full family; added GLM-4.7, Kimi K2.5, MiMo-V2-Flash; Claude 4.6 1M context at standard pricing |

---

_Pricing and capabilities change rapidly — always validate against official documentation before production deployment._  
_Sources: [Anthropic Pricing](https://docs.anthropic.com/en/docs/about-claude/models/overview) · [OpenAI Pricing](https://platform.openai.com/docs/pricing) · [Google AI Pricing](https://ai.google.dev/pricing)_
