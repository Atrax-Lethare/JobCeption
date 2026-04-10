# AI / Intelligence Spec

- **Version:** 0.2
- **Status:** Draft
- **Owner:** AI / ML
- **Last Updated:** 2026-04-10
- **Depends On:** 02-domain-model.md, 04-privacy-and-compliance.md
- **Referenced By:** 03-evaluation-spec.md, 03-recommendation-and-search-spec.md, 03-sandbox-execution-spec.md, 09-trust-and-reputation-spec.md

## Purpose
Define how JobCeption uses AI and ML, what models are used where, what guardrails apply, how evals are run, and what we will not use AI for. JobCeption's AI strategy is **free and open-source first**: Hugging Face models (self-hosted on CPU where possible) and OSS libraries are the default; hosted commercial LLMs (Anthropic, OpenAI) are an explicit fallback used only where free options demonstrably underperform.

## Strategy
1. **Free / open-source first.** Every AI use case is implemented with a Hugging Face model or an OSS library before any hosted API is considered.
2. **Self-host the small models.** Sentence transformers, classifiers, and detectors run on our own CPU instances to avoid free-tier rate limits and to keep latency predictable.
3. **HF Inference API for the larger models.** Mistral-7B, BART-large, and similar are called via the HF Inference API (free tier at MVP, paid Inference Endpoints if and when scale demands).
4. **Hosted LLMs are a fallback, not a default.** Anthropic and OpenAI are wired into the AI Gateway as last-resort providers for cases where free models demonstrably underperform (subtle moderation, complex dispute summarization). They are off by default and gated behind a feature flag.
5. **Rank, never reject.** AI outputs rank, score, and triage; humans make terminal decisions.

## AI Gateway
A single internal service (`ai-gateway`) fronts every model call. Responsibilities:
- Provider routing: HF self-hosted → HF Inference API → hosted LLM fallback (off by default).
- PII stripping at the boundary.
- Token / call budget enforcement per use case.
- Prompt versioning (every prompt has a hash and ID; prompts are code, reviewed in PR).
- Per-use-case fallback chains.
- Metrics, traces, and cost logging.

The gateway is the only component allowed to call external AI vendors. No service calls Anthropic, OpenAI, or HF directly.

## AI Use Cases — Model Map

| # | Use Case | Primary Model / Tool | Hosting | Fallback |
|---|---|---|---|---|
| 1 | Bounty recommendation (content vectors) | `sentence-transformers/all-MiniLM-L6-v2` | Self-hosted CPU | — |
| 2 | Bounty recommendation (collaborative filter) | `LightFM` (Python OSS) | Self-hosted | `Surprise` lib |
| 3 | Recruiter semantic search | `sentence-transformers/all-MiniLM-L6-v2` + Weaviate | Self-hosted | Pinecone free tier |
| 4 | Candidate ↔ JD matching | `cross-encoder/ms-marco-MiniLM-L-6-v2` | Self-hosted CPU | — |
| 5 | Code submission AI-content detection | `roberta-base-openai-detector` | Self-hosted CPU | — |
| 6 | Post AI-slop detection | `roberta-base-openai-detector` + heuristics | Self-hosted CPU | — |
| 7 | Plagiarism (code) | MOSS algorithm + MinHash/LSH | Self-hosted | — |
| 8 | Plagiarism (text) | sentence-transformers cosine | Self-hosted | — |
| 9 | Writing quality scoring | `facebook/bart-large-mnli` (zero-shot) | HF Inference API | self-hosted DistilBART |
| 10 | Grammar / style | LanguageTool API (free OSS) | Self-hosted | — |
| 11 | Design output evaluation | `openai/clip-vit-base-patch32` | HF Inference API | self-hosted CLIP |
| 12 | Portfolio summarization | `facebook/bart-large-cnn` | HF Inference API | `mistralai/Mistral-7B-Instruct` |
| 13 | Bounty difficulty calibration | Statistical (no ML) + LLM review on edge cases | Self-hosted + HF | hosted LLM (gated) |
| 14 | Moderation triage | `facebook/bart-large-mnli` zero-shot | HF Inference API | hosted LLM (gated) |
| 15 | Search relevance reranking | `cross-encoder/ms-marco-MiniLM-L-6-v2` | Self-hosted CPU | — |

Detailed pipelines for each use case live in the relevant feature spec — see `03-evaluation-spec.md`, `03-recommendation-and-search-spec.md`, `09-trust-and-reputation-spec.md` (fraud), and `09-social-spec.md` (slop).

## Model Hosting Strategy

### Self-hosted (CPU)
The following models run on our own CPU instances inside the EKS cluster, served via FastAPI wrappers behind the AI Gateway:
- `sentence-transformers/all-MiniLM-L6-v2` (≈80MB, ~50ms inference on CPU)
- `cross-encoder/ms-marco-MiniLM-L-6-v2` (≈80MB)
- `roberta-base-openai-detector` (≈500MB)
- `openai/clip-vit-base-patch32` (≈600MB) — moved here once volume justifies
- LanguageTool server

These are cheap to operate, have no rate limits, and give predictable latency. Each model is a separate Kubernetes Deployment with HPA.

### HF Inference API (free tier at MVP)
The larger models are called via HF Inference API at MVP:
- `facebook/bart-large-mnli`
- `facebook/bart-large-cnn`
- `mistralai/Mistral-7B-Instruct` (when needed)

**Limits to be aware of:**
- HF Inference API free tier is rate-limited; expect occasional 429s.
- Cold-start latency on free tier can be 10–30s for less-popular models.
- Production paths must tolerate these failures via fallback to a self-hosted variant or graceful degradation.

When MVP usage outgrows the free tier, the migration path is **HF Inference Endpoints** (paid, dedicated hardware, no rate limits) — NOT a switch to OpenAI/Anthropic.

### Hosted LLMs (fallback, gated)
Anthropic and OpenAI clients exist in the AI Gateway but are **disabled by default via feature flag**. They are turned on for a specific use case only when:
1. Offline eval shows the free model underperforms below threshold.
2. The use case is low-volume (so cost is bounded).
3. Tech Lead and Finance approve the budget line.

Currently anticipated uses (all gated, none default-on):
- Subtle moderation cases the BART zero-shot model marks as ambiguous.
- Complex dispute summarization for admin review.

## Prompts
- Versioned in source control under `ai/prompts/<use_case>/v<n>.txt`.
- Every prompt has a hash and a stable ID; the gateway logs which version was used per call.
- Prompt edits go through PR review.
- Reverting a prompt is a code revert, not a database change.

## Guardrails
- **No automated rejection.** No code path routes a candidate to a negative terminal state based on an AI output alone.
- **No PII to external models.** The gateway strips emails, phone numbers, government IDs, and full names before calling any external API.
- **No training on user data.** HF Inference API and any hosted LLM provider are contractually bound not to train on submitted data; verified annually.
- **Bias monitoring.** Quarterly audits of recommendation and ranking outputs by declared demographic segment.
- **Human in the loop on every score that affects reputation.** Platform evaluations may use AI as a signal but always have a deterministic non-AI component too.
- **Forbidden uses:** generating fake bounty content, generating fake reviews, scoring candidates on demographic features, mimicking real users in Connections, auto-rejecting candidates.

## Evaluations

### Offline
Every model in production has a golden dataset and explicit precision/recall targets:

| Model | Metric | Target |
|---|---|---|
| AI-content detector | Precision | ≥ 0.90 |
| AI-content detector | Recall | ≥ 0.85 |
| AI-slop detector | Precision | ≥ 0.90 |
| Plagiarism (code, MOSS+MinHash) | Recall on known-duplicate set | ≥ 0.95 |
| Cross-encoder (JD ↔ candidate) | nDCG@10 vs. labeled set | ≥ 0.80 |
| Bounty recommender | nDCG@10 | ≥ 0.65 |

A model that fails its eval cannot ship.

### Online
A/B tests are gated behind feature flags. Success metrics live in `09-analytics-spec.md`. Recommendation and reranking changes always go through online A/B before full rollout.

### Red team
Before any model swap, an adversarial pass is run: known cheating patterns against the detectors, known adversarial inputs against classifiers, prompt injection attempts against any LLM-mediated path.

## Fallback Chains
If the primary model is unavailable, the gateway falls back in order. The system never blocks a user-facing action on AI availability.

| Use Case | Primary | Fallback 1 | Fallback 2 |
|---|---|---|---|
| Bounty recommendation | LightFM + ST | Content-only (ST cosine) | Recency sort |
| Recruiter semantic search | ST + Weaviate | Postgres trigram search | Tag filter only |
| AI-content detection | RoBERTa detector | Heuristic (length, perplexity) | Pass-through (no flag) |
| Plagiarism | MOSS + MinHash | MinHash only | Deferred to admin queue |
| Portfolio summarization | BART-large-cnn (HF) | Mistral-7B (HF) | Template-based summary |
| Moderation triage | BART zero-shot (HF) | Heuristic ranking | Manual triage only |

## Cost Model
- **Self-hosted models:** flat monthly compute cost on EKS (modest — these are CPU workloads).
- **HF Inference API:** free tier at MVP; budgeted line item in Phase 2 if volume warrants Inference Endpoints.
- **Hosted LLMs:** zero by default; only billed when a feature flag is enabled and approved.
- **Vector DB (Weaviate self-hosted):** flat compute.

This cost profile is the primary reason for the strategy — it makes JobCeption economically viable as a small-team build.

## Open Questions
- Should we self-host BART-large or stay on HF Inference API even at scale?
- Where is the threshold at which Inference Endpoints beats self-hosting on TCO?
- Do we need a vector database refresh strategy as embedding models are updated?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Full rewrite. Pivoted from hosted-LLM-first to Hugging Face / free OSS first. Hosted LLMs demoted to gated fallback. Added explicit model map, hosting tiers, fallback chains, and offline eval targets.
