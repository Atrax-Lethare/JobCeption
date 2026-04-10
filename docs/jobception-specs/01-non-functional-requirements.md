# Non-Functional Requirements

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Tech Lead
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 07-scalability-spec.md, 07-observability-spec.md, 08-testing-strategy.md

## Purpose
Performance budgets, availability targets, and quality attributes the system must satisfy.

## Performance Budgets
- **API median latency:** ≤ 200 ms (read), ≤ 400 ms (write).
- **API p99 latency:** ≤ 1 s.
- **Web LCP:** ≤ 2.5 s on 4G.
- **Sandbox cold-start:** ≤ 3 s (warm pool target).
- **Sandbox max execution time per submission:** configurable per bounty, default 60 min.

## Availability & Reliability
- **Platform uptime SLO:** 99.9% monthly (≈ 43 min downtime).
- **Sandbox subsystem SLO:** 99.5% (degraded mode allows queued submissions).
- **Pledge service SLO:** 99.95% (legally significant; cannot drop writes).
- **Data durability:** 11 nines (object storage) for submissions and pledges.

## Scalability Targets
- **Concurrent active bounty sessions at launch:** 1,000.
- **At year 1:** 10,000.
- **Peak (sponsored hackathon):** 50,000.
- **Post-write read replication lag:** < 1 s.

## Security
- All data encrypted in transit (TLS 1.3) and at rest.
- Sandbox workers fully isolated, no outbound network unless whitelisted.
- Secrets via vault; no plaintext in repo or env files.
- See `04-security-spec.md`.

## Privacy
- Compliance with DPDP (India), GDPR (EU), CCPA (CA).
- User data export and deletion within 30 days.
- See `04-privacy-and-compliance.md`.

## Accessibility
- WCAG 2.2 AA across all candidate-facing surfaces.
- See `05-accessibility-spec.md`.

## Observability
- All requests traced.
- Pledge events logged immutably for legal recall.
- See `07-observability-spec.md`.

## Maintainability
- Test coverage ≥ 80% lines on backend, ≥ 70% on frontend.
- Every public API endpoint documented in OpenAPI.

## Open Questions
- Do we need multi-region writes at launch or single-region with read replicas?
- What's the acceptable cold-start latency ceiling for the sandbox before users abandon?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
