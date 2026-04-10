# Integration Spec

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Backend
- **Last Updated:** 2026-04-10
- **Depends On:** —
- **Referenced By:** —

## Purpose
Catalogue every third-party integration with its purpose, contract, and fallback. Reflects JobCeption's free-and-open-source-first philosophy: hosted commercial APIs are minimized in favor of self-hosted OSS where viable.

## Code Execution & Sandboxing

### Judge0
- **Purpose:** code execution sandbox for code-challenge bounties. See `03-sandbox-execution-spec.md`.
- **Hosting:** self-hosted on EKS (open source).
- **Contract:** REST API; orchestrator submits source + language + stdin + limits, receives stdout/stderr/time/memory.
- **Fallback:** none — Judge0 IS the runtime for code bounties. If Judge0 is down, code-bounty submissions queue.

### GitHub Codespaces
- **Purpose:** full-IDE environments for project-style bounties.
- **Hosting:** GitHub-managed.
- **Contract:** GitHub REST API for codespace lifecycle; OAuth scoped to a JobCeption GitHub org.
- **Fallback:** StackBlitz for compatible stacks; otherwise, the bounty pauses with a user-facing notice.
- **Cost:** per-minute billing — orchestrator enforces caps.

### StackBlitz SDK
- **Purpose:** in-browser WebContainer-based runtimes for Node/frontend bounties.
- **Hosting:** runs in the candidate's browser; no server cost.
- **Contract:** JS SDK embedded in the bounty page.
- **Fallback:** Codespaces.

## AI / ML

### Hugging Face Inference API
- **Purpose:** inference for larger models (BART-large, Mistral-7B, CLIP).
- **Hosting:** HF-managed.
- **Contract:** REST API via the AI Gateway.
- **Free tier limits:** rate-limited; cold-start latency on less-popular models.
- **Fallback:** self-hosted variant of the same model family if available; else, graceful degradation per the AI spec.
- **Migration path at scale:** HF Inference Endpoints (paid, dedicated hardware).

### Self-hosted Hugging Face Models
- **Purpose:** small models that run on CPU (sentence-transformers, cross-encoder, RoBERTa detector).
- **Hosting:** FastAPI wrappers on EKS, behind the AI Gateway.
- **Contract:** internal HTTP.
- **Fallback:** see per-model fallback chains in `03-ai-intelligence-spec.md`.

### Anthropic API (gated fallback)
- **Purpose:** subtle moderation cases, complex dispute summarization. **Off by default.**
- **Hosting:** Anthropic-managed.
- **Contract:** via AI Gateway only, behind a feature flag.

### OpenAI API (gated fallback)
- **Purpose:** alternate hosted LLM provider. **Off by default.**
- **Hosting:** OpenAI-managed.
- **Contract:** via AI Gateway only, behind a feature flag.

## Code Quality & Plagiarism

### SonarQube (Community Edition)
- **Purpose:** static analysis on code submissions — bugs, code smells, security issues, quality score.
- **Hosting:** self-hosted on EKS.
- **Contract:** REST API; orchestrator submits source, retrieves analysis report.
- **Fallback:** rule-based linters per language (eslint, pylint, etc.).

### MOSS (Stanford Measure Of Software Similarity)
- **Purpose:** code plagiarism detection across submissions to the same bounty.
- **Hosting:** open-source algorithm; reimplemented in-house OR scripted via the public MOSS service.
- **Contract:** batch job after submission.
- **Fallback:** MinHash/LSH-only mode (less precise but always available).

### LanguageTool
- **Purpose:** grammar and style for writing-bounty submissions.
- **Hosting:** self-hosted (open source).
- **Contract:** REST API.
- **Fallback:** disable grammar scoring for the affected submissions; flag for human review.

## Vector Database

### Weaviate (self-hosted)
- **Purpose:** vector store for recruiter semantic search and bounty recommendation.
- **Hosting:** self-hosted on EKS.
- **Contract:** REST + GraphQL.
- **Fallback:** Pinecone free tier (read-only mirror as DR).

## Payments & Escrow

### Razorpay (India primary)
- **Purpose:** payments and escrow for freelance bounties in INR.
- **Hosting:** Razorpay-managed.
- **Contract:** REST API + webhooks.
- **Fallback:** queue + manual reconciliation if down.

### Stripe Connect (international)
- **Purpose:** payments and escrow for non-INR freelance bounties.
- **Hosting:** Stripe-managed.
- **Contract:** REST API + webhooks.
- **Fallback:** queue + manual reconciliation.

## Identity Verification (KYC)

### Persona / Onfido / Hyperverge
- **Purpose:** government ID + liveness check for the Verified Identity badge and high-value freelance bounties.
- **Hosting:** vendor-managed.
- **Contract:** REST API + webhooks.
- **Fallback:** queue + retry; user blocked from issuing pledges until verified.
- **Choice:** TBD by ADR; Hyperverge for India-only launch is the leading candidate (cost).

## Communications

### Email — Postmark (primary), AWS SES (fallback)
### SMS — MSG91 (India), Twilio (rest of world)

## Calendar Integration

### Google Calendar API + Microsoft Graph
- **Purpose:** two-way sync of pipeline stages and interview slots.
- **Contract:** OAuth.

## Object Storage & Search

### AWS S3
- **Purpose:** submissions, attachments, documents, backups.
- **Region:** ap-south-1.

### Meilisearch (self-hosted)
- **Purpose:** full-text search across the marketplace and Connections feed.
- **Hosting:** self-hosted on EKS.

## Observability

### Sentry (errors)
### Grafana / Prometheus / Loki / Tempo (metrics, logs, traces)
- All self-hosted or Grafana Cloud free tier.

## Webhook Contract Conventions
- HMAC-SHA256 signature in `X-JobCeption-Signature`.
- Replay protection via `X-JobCeption-Timestamp` (5 min window).
- Versioned payloads.

## Open Questions
- Whether to use the public MOSS service or reimplement the algorithm in-house.
- KYC vendor selection (cost vs. quality vs. India-specific support).
- Whether to dual-run Razorpay and Stripe from day one, or single-provider at MVP.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Full rewrite. Reorganized around free/OSS-first stack: Judge0, self-hosted HF, Weaviate, SonarQube CE, MOSS, LanguageTool. Hosted LLMs demoted to gated fallback. Razorpay promoted as India-primary.
