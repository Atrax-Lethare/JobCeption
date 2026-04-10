# Data Governance

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Data / Legal
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 02-database-spec.md, 04-privacy-and-compliance.md, 09-analytics-spec.md

## Purpose
Define how data is classified, where it can live, who owns it, and how it is retained, deleted, and lineaged.

## Data Classification
| Class | Examples | Handling |
|-------|----------|----------|
| **C0 — Public** | Bounty titles, post bodies | No restriction |
| **C1 — Internal** | Aggregate metrics, telemetry | Internal access only |
| **C2 — Confidential** | Profile details, salary ranges, evaluations | RBAC required, encrypted at rest |
| **C3 — Sensitive** | Government IDs, KYC images, payment data | Tokenized, isolated store, restricted access |
| **C4 — Legally Privileged** | Pledges, dispute evidence | Immutable, audit-logged, 7-year retention |

## Data Residency
- India-launch: data stored in `ap-south-1` (Mumbai) region.
- EU users: data stored in EU region; replication blocked by classification policy.
- Cross-region transfers require explicit DPA coverage.

## Right to Erasure
- User triggers deletion via Settings → "Delete account".
- Within 30 days:
  - C0–C2 data anonymized (FK references replaced with `[deleted_user]` placeholder).
  - C3 data hard-deleted.
  - C4 data retained but PII fields tombstoned (legal requirement).
- Reputation events affecting third parties are preserved without PII.

## Lineage
Every analytics event must declare its source table and the transformation pipeline applied. Tracked in a lineage manifest checked into the repo.

## Ownership
- Identity, profile, KYC: Security team.
- Bounty, submission, evaluation: Backend team.
- Pledge, dispute, audit: Tech Lead + Legal.
- Connections, posts, badges: Product team.

## Open Questions
- How do we handle a deleted user who is still referenced in active disputes?
- Are we comfortable retaining anonymized reputation history forever, or do we need a hard cutoff?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
