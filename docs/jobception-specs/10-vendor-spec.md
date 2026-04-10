# Vendor Spec

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Tech Lead
- **Last Updated:** 2026-04-10
- **Depends On:** —
- **Referenced By:** 03-integration-spec.md, 03-ai-intelligence-spec.md

## Purpose
Inventory of third-party dependencies — purpose, criticality, cost model, fallback, and license. Reflects JobCeption's free-OSS-first philosophy.

## Vendor Inventory

### Critical Infrastructure
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| AWS | Cloud infra | Critical | Pay-as-you-go | N/A (lock-in) | Commercial |
| GitHub | Source + Codespaces + actions | Critical | Per-seat + per-minute | GitLab fallback | Commercial |

### Code Execution & Sandboxing
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| Judge0 | Code execution sandbox | Critical | Free (self-hosted compute only) | None — IS the runtime | OSS (MIT) |
| GitHub Codespaces | Project IDE | High | Per-minute | StackBlitz | Commercial |
| StackBlitz SDK | Browser WebContainers | Medium | Free SDK | Codespaces | Commercial (free SDK) |

### AI / ML — Self-Hosted
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| Hugging Face (model weights) | Model source | High | Free | Mirrored locally | Various OSS |
| sentence-transformers | Embeddings | High | Free (compute only) | — | OSS |
| LightFM | Recommender | Medium | Free | Surprise lib | OSS |
| LanguageTool | Grammar/style | Medium | Free (self-hosted) | Per-language linters | OSS (LGPL) |
| MOSS | Code plagiarism | High | Free | MinHash/LSH only | Free academic |
| SonarQube CE | Code quality | Medium | Free | Rule-based linters | OSS |
| Weaviate | Vector DB | High | Free (self-hosted) | Pinecone free tier | OSS (BSD) |

### AI / ML — Managed
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| Hugging Face Inference API | Larger model inference | Medium | Free tier → paid Endpoints | Self-hosted variant | Commercial (free tier) |
| Anthropic API | LLM (gated fallback only) | Low | Pay-per-token, off by default | Disabled | Commercial |
| OpenAI API | LLM (gated fallback only) | Low | Pay-per-token, off by default | Disabled | Commercial |

### Payments & KYC
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| Razorpay | India payments + escrow | Critical | % of TPV | Stripe queue | Commercial |
| Stripe Connect | Global payments + escrow | High | % of TPV | Razorpay | Commercial |
| Persona / Onfido / Hyperverge | KYC | High | Per-verification | Queue + retry | Commercial |

### Communications
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| Postmark | Email (primary) | Medium | Per-email | SES | Commercial |
| AWS SES | Email (fallback) | Medium | Per-email | Postmark | Commercial |
| MSG91 | SMS (India) | Medium | Per-SMS | Twilio | Commercial |
| Twilio | SMS (global) | Medium | Per-SMS | MSG91 | Commercial |

### Search & Storage
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| Meilisearch | Full-text search | High | Free (self-hosted) | Postgres FTS | OSS |
| AWS S3 | Object storage | Critical | Pay-as-you-go | — | Commercial |

### Observability
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| Sentry | Error tracking | Low | Free tier → paid | Self-hosted Sentry | OSS-core |
| Grafana / Prom / Loki / Tempo | Metrics, logs, traces | Medium | Self-hosted free | Grafana Cloud free | OSS |

### Database
| Vendor | Purpose | Criticality | Cost | Fallback | License |
|---|---|---|---|---|---|
| PostgreSQL | Primary DB | Critical | Free | — | OSS |
| Redis | Cache + queue | High | Free | — | OSS |

## Cost Profile Summary
The vendor strategy is deliberately tilted toward zero-marginal-cost OSS for AI/ML workloads:
- **AI/ML stack:** ~$0 baseline (just compute) at MVP. Anthropic/OpenAI line items remain at $0 unless explicitly enabled.
- **Sandboxing:** Judge0 self-hosted = $0 software cost. Codespaces is per-minute, capped per bounty.
- **Vector DB:** Weaviate self-hosted = $0 software cost.
- **The largest cost lines** are: AWS infra, GitHub Codespaces minutes, Razorpay/Stripe transaction fees, and KYC per-verification fees.

This profile is the reason JobCeption is buildable as a small-team product. It is also a constraint: every vendor change that adds a paid dependency requires explicit Tech Lead + Finance approval.

## Vendor Review Cadence
- Annual review of every Critical and High vendor.
- DPA on file for every vendor handling user data.
- SOC2 / ISO 27001 verified where applicable.

## Vendor Risk
- Single-vendor lock-in flagged in the inventory.
- Each Critical vendor has a documented switch-out plan.
- AWS is the deepest lock-in; Codespaces is the second.

## License Compliance
- All OSS licenses tracked via FOSSA or equivalent.
- AGPL forbidden in client code.
- LGPL (LanguageTool) — used as a separate process via REST, no static linking, compliant.

## Open Questions
- HF Inference Endpoints budget threshold — at what HF API call volume do we switch?
- Whether to mirror HF model weights to S3 to insulate against HF availability.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Full rewrite. Inventory reorganized to highlight free/OSS-first stack. Added Judge0, Codespaces, StackBlitz, Weaviate, LightFM, LanguageTool, MOSS, SonarQube. Demoted Anthropic/OpenAI to gated fallback. Added cost profile summary.
