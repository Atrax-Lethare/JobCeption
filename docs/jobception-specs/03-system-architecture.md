# System Architecture

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Tech Lead
- **Last Updated:** 2026-04-10
- **Depends On:** —
- **Referenced By:** 03-tech-stack.md, 03-api-spec.md, 03-sandbox-execution-spec.md, 07-infrastructure-spec.md

## Purpose
High-level service decomposition, data flow, and synchronous vs. asynchronous boundaries.

## Architecture Style
Modular monolith for MVP → carve out hot services as scale demands. Specifically: extract Sandbox Workers and Pledge Service first if needed.

## Services (logical)
1. **API Gateway / BFF** — single entrypoint for web client; auth, rate limiting, request validation.
2. **Identity Service** — users, companies, sessions, KYC.
3. **Bounty Service** — CRUD, templates, constraints, marketplace queries.
4. **Sandbox Orchestrator** — provisions workers, streams events, accepts submissions. See `03-sandbox-execution-spec.md`.
5. **Sandbox Worker (fleet)** — isolated execution; one container per session.
6. **Evaluation Service** — platform-side scoring, feeds into Reputation.
7. **Pipeline Service** — hiring pipelines, stages, document requests.
8. **Pledge Service** — issues and stores pledges; legally critical, isolated for SLO.
9. **Reputation Service** — append-only event ledger, materialized scores.
10. **Connections Service** — posts, feed, badges, visibility scoring.
11. **Notification Service** — email, in-app, push, real-time WebSockets.
12. **Escrow Service** — interfaces to Razorpay (India primary) / Stripe Connect.
13. **Admin Service** — moderation, dispute resolution, support.
14. **Analytics Pipeline** — event ingestion, ETL.
15. **AI Gateway** — single egress point for all model calls; routes between self-hosted HF models, HF Inference API, and (gated) hosted LLMs. See `03-ai-intelligence-spec.md`.
16. **Recommendation & Search Service** — bounty recommendation, recruiter semantic search, JD↔candidate matching. Uses Weaviate. See `03-recommendation-and-search-spec.md`.
17. **Fraud & Integrity Service** — plagiarism detection (MOSS + MinHash/LSH), behavioral biometrics, AI-content detection, identity-tied risk scoring. Feeds the Reputation and Admin services.

## Data Flow Highlights
- **Bounty publish:** Bounty Service writes → Search Indexer (async) → Marketplace queries hit Search.
- **Submission:** Sandbox Runtime (Judge0 / Codespaces / Docker) → Sandbox Orchestrator → Evaluation Service (per-type pipeline, see `03-evaluation-spec.md`) → Reputation Service → Materialized XP.
- **AI calls:** every model call (embedding, classification, scoring, summarization) goes through the AI Gateway. No service calls Anthropic, OpenAI, or HF directly.
- **Pledge:** Pipeline Service → Pledge Service (sync, with idempotency key) → Audit Log → Notification Service.
- **Review:** Pipeline Service → Reputation Service (async).

## Sync vs. Async
- **Sync:** anything user-blocking, anything legally significant (pledges).
- **Async:** indexing, notifications, analytics, reputation materialization.
- Message bus: Redis Streams or NATS for MVP; Kafka if volume justifies.

## Failure Domains
- Sandbox subsystem failure must not bring down marketplace browsing.
- Reputation Service failure must not block Pledge writes.
- Notification failure must never block the originating action.

## Open Questions
- Adopt event sourcing for reputation, or stick with append-only ledger + materialization?
- Single region or multi-region active-active for the pledge service?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Added AI Gateway, Recommendation & Search Service, and Fraud & Integrity Service. Updated submission flow to reference per-type evaluation pipelines.
