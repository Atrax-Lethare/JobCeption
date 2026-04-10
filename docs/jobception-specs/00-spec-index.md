# Specification Index

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Tech Lead
- **Last Updated:** 2026-04-10
- **Depends On:** —
- **Referenced By:** —

## Purpose
Master index for all JobCeption specifications. Defines what specs exist, their precedence, ownership, and how they relate. Any AI coding agent or human contributor MUST read this file first, followed by `00-glossary.md` and `01-product-overview.md`.

## How to Use This Framework
1. **Read top-down.** Start at `00-spec-index.md`, then `00-glossary.md`, then `01-product-overview.md`. Everything else assumes you've read these three.
2. **Match weight to stakes.** Trim ruthlessly. Specs are not novels.
3. **One source of truth per concept.** Higher-precedence specs win conflicts. Update the loser immediately.
4. **Specs are living documents.** Every change is reflected in the spec's own Change Log and, if significant, captured as an ADR in `00-architecture-decision-records/`.

## Precedence Rules
1. **Legal & Compliance** — `04-privacy-and-compliance.md`, `04-legal-spec.md`
2. **Security** — `04-security-spec.md`, `04-auth-spec.md`, `04-authorization-spec.md`
3. **Product Foundation** — `01-*`
4. **Domain & Data** — `02-*`
5. **Architecture & APIs** — `03-*`
6. **Infrastructure & Operations** — `07-*`
7. **Design & Experience** — `05-*`, `06-*`
8. Everything else

## Spec Catalogue

### 00 — Meta & Governance
| File | Purpose | Owner |
|------|---------|-------|
| `00-spec-index.md` | This file. | Tech Lead |
| `00-glossary.md` | Canonical terms. | Product |
| `00-architecture-decision-records/` | ADRs. | Tech Lead |

### 01 — Product Foundation
| File | Purpose | Owner |
|------|---------|-------|
| `01-product-overview.md` | Vision, problem, users, metrics. | Product |
| `01-personas-and-journeys.md` | Personas, JTBD, journey maps. | Product / Design |
| `01-functional-requirements.md` | Features, stories, acceptance criteria. | Product |
| `01-non-functional-requirements.md` | SLOs, perf budgets, availability. | Tech Lead |
| `01-scope-and-constraints.md` | In/out of scope, constraints. | Product |

### 02 — Domain & Data
| File | Purpose | Owner |
|------|---------|-------|
| `02-domain-model.md` | Entities, relationships, invariants, lifecycles. | Tech Lead |
| `02-database-spec.md` | Schema, indexing, migrations, retention. | Backend |
| `02-data-governance.md` | Classification, residency, lineage, deletion. | Data / Legal |

### 03 — Architecture & Tech
| File | Purpose | Owner |
|------|---------|-------|
| `03-system-architecture.md` | High-level diagram, services, data flow. | Tech Lead |
| `03-tech-stack.md` | Languages, frameworks, versions, rationale. | Tech Lead |
| `03-api-spec.md` | API contracts, versioning, errors. | Backend |
| `03-integration-spec.md` | Third-party services, webhooks, events. | Backend |
| `03-ai-intelligence-spec.md` | Models, prompts, evals, guardrails. | AI / ML |
| `03-sandbox-execution-spec.md` | **NEW** — bounty sandbox runtime (Judge0 + Codespaces + Docker). | Tech Lead / Security |
| `03-evaluation-spec.md` | **NEW** — per-bounty-type evaluation pipelines. | Tech Lead / AI |
| `03-recommendation-and-search-spec.md` | **NEW** — bounty recommendation, recruiter search, JD↔candidate matching. | AI / Backend |

### 04 — Security, Identity & Compliance
| File | Purpose | Owner |
|------|---------|-------|
| `04-security-spec.md` | Threat model, encryption, secrets. | Security |
| `04-auth-spec.md` | Auth flows, sessions, MFA, passkeys. | Security / Backend |
| `04-authorization-spec.md` | RBAC/ABAC, permission matrix. | Security / Backend |
| `04-privacy-and-compliance.md` | GDPR, DPDP, CCPA. | Legal |
| `04-legal-spec.md` | ToS, Privacy Policy, pledges as legal artifacts. | Legal |

### 05 — Design & Experience
| File | Purpose | Owner |
|------|---------|-------|
| `05-design-system.md` | Tokens, components, type, color, spacing. | Design |
| `05-ui-ux-spec.md` | Screens, flows, states, microcopy. | Design |
| `05-accessibility-spec.md` | WCAG target, keyboard, SR, contrast. | Design |

### 06 — Frontend & Client
| File | Purpose | Owner |
|------|---------|-------|
| `06-frontend-spec.md` | Routing, state, rendering. | Frontend |
| `06-error-handling-spec.md` | Client errors, retries, fallbacks. | Frontend |

### 07 — Infrastructure & Operations
| File | Purpose | Owner |
|------|---------|-------|
| `07-infrastructure-spec.md` | Cloud, regions, networking, IaC. | DevOps |
| `07-containerization-spec.md` | Docker, orchestration, image policy. | DevOps |
| `07-environments-spec.md` | Dev/staging/prod, secrets, flags. | DevOps |
| `07-scalability-spec.md` | LB, caching, queues. | Tech Lead |
| `07-devops-spec.md` | CI/CD, branching, releases. | DevOps |
| `07-observability-spec.md` | Logs, metrics, traces, alerts. | DevOps |

### 08 — Quality & Testing
| File | Purpose | Owner |
|------|---------|-------|
| `08-testing-strategy.md` | Unit, integration, e2e, load, coverage. | QA / Tech Lead |

### 09 — Growth, Engagement & Lifecycle
| File | Purpose | Owner |
|------|---------|-------|
| `09-analytics-spec.md` | Events, funnels, retention. | Product / Data |
| `09-social-spec.md` | Connections feed, posts, badges, notifications. | Product / Backend |
| `09-monetization-spec.md` | Pricing, billing, escrow fees. | Product / Finance |
| `09-admin-and-internal-tools.md` | Moderation, support, dispute UI. | Ops |
| `09-trust-and-reputation-spec.md` | **NEW** — XP formulas, pledges, ratings. | Product / Tech Lead |

### 10 — Lifecycle Management
| File | Purpose | Owner |
|------|---------|-------|
| `10-deployment-spec.md` | Targets, blue/green, canary. | DevOps |
| `10-vendor-spec.md` | Third-party deps, SLAs, fallbacks. | Tech Lead |

## Cross-Reference Map (key dependencies)
- `02-domain-model.md` → `02-database-spec.md`, `03-api-spec.md`, `03-ai-intelligence-spec.md`, `09-analytics-spec.md`, `09-trust-and-reputation-spec.md`
- `09-trust-and-reputation-spec.md` → `02-domain-model.md`, `03-api-spec.md`, `04-legal-spec.md`, `09-monetization-spec.md`, `09-admin-and-internal-tools.md`
- `03-sandbox-execution-spec.md` → `03-system-architecture.md`, `04-security-spec.md`, `07-containerization-spec.md`, `07-scalability-spec.md`, `03-evaluation-spec.md`
- `03-evaluation-spec.md` → `02-domain-model.md`, `03-ai-intelligence-spec.md`, `03-sandbox-execution-spec.md`, `09-trust-and-reputation-spec.md`
- `03-recommendation-and-search-spec.md` → `02-domain-model.md`, `03-ai-intelligence-spec.md`, `09-analytics-spec.md`
- `04-authorization-spec.md` → `03-api-spec.md`, `06-frontend-spec.md`, `09-admin-and-internal-tools.md`
- `05-design-system.md` → `05-ui-ux-spec.md`, `06-frontend-spec.md`
- `01-non-functional-requirements.md` → `07-scalability-spec.md`, `07-observability-spec.md`, `08-testing-strategy.md`
- `02-data-governance.md` → `04-privacy-and-compliance.md`, `02-database-spec.md`, `09-analytics-spec.md`

## Spec Template
```markdown
# [Spec Name]
- **Version:** 0.2
- **Status:** Draft | Review | Approved | Deprecated
- **Owner:** [Role]
- **Last Updated:** YYYY-MM-DD
- **Depends On:** [list]
- **Referenced By:** [list]

## Purpose
One paragraph.

## Scope
What's in. What's out.

## Specification
The actual content.

## Open Questions
Unresolved items.

## Change Log
- YYYY-MM-DD — v0.1 — Initial draft.
```

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Added 03-evaluation-spec.md and 03-recommendation-and-search-spec.md to catalogue. Updated cross-references.
