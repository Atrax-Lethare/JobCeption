# Tech Stack

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Tech Lead
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** —

## Purpose
Concrete technology choices and the rationale behind each. Locks down what the team builds with.

## Frontend
- **Framework:** Next.js 14+ (App Router), React 18.
- **Language:** TypeScript (strict mode).
- **Styling:** Tailwind CSS + a small in-house component library.
- **State:** TanStack Query for server state, Zustand for UI state.
- **Forms:** React Hook Form + Zod.
- **Rationale:** mature ecosystem, SSR for SEO on company/bounty pages, strong typing end-to-end.

## Backend
- **Language:** TypeScript (Node.js 20) OR Go 1.22 (decide via ADR-0002).
- **API style:** REST + JSON for MVP; GraphQL gateway considered post-MVP.
- **Framework:** NestJS (if TS) or Chi/Echo (if Go).
- **Validation:** Zod (TS) or go-playground/validator (Go).

## Sandbox Runtime
- **Container:** Docker images per language.
- **Isolation:** Firecracker microVMs (preferred) or gVisor.
- **Orchestration:** Custom orchestrator on Kubernetes.

## Data
- **Primary DB:** PostgreSQL 16.
- **Search:** Meilisearch.
- **Cache & queues:** Redis 7.
- **Object storage:** S3 (or compatible).

## AI / ML
- **Hosting:** managed inference (Anthropic, OpenAI) for MVP.
- **Plagiarism / AI-slop classifiers:** small fine-tuned models, self-hosted.

## Infrastructure
- **Cloud:** AWS (primary), region `ap-south-1`.
- **IaC:** Terraform.
- **Container orchestration:** EKS.
- **CI/CD:** GitHub Actions.
- **Secrets:** AWS Secrets Manager.

## Observability
- **Logs:** structured JSON → CloudWatch + Loki.
- **Metrics:** Prometheus + Grafana.
- **Tracing:** OpenTelemetry → Tempo.
- **Errors:** Sentry.

## Email & SMS
- Email: Postmark or SES.
- SMS: MSG91 (India) / Twilio (global).

## Open Questions
- TS vs. Go for backend (ADR-0002).
- Self-hosted Postgres vs. managed (RDS / Aurora).

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
