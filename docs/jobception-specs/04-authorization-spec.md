# Authorization Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Security / Backend
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 03-api-spec.md, 06-frontend-spec.md, 09-admin-and-internal-tools.md

## Purpose
Define the permission model — roles, attributes, and the matrix of who can do what.

## Model
Hybrid RBAC + ABAC:
- **Roles** for coarse control (Candidate, Recruiter, Hiring Manager, Company Owner, Admin, Moderator, Support).
- **Attributes** for fine control (e.g. `bounty.company_id == user.company_id`).

## Core Roles
| Role | Scope | Notes |
|------|-------|-------|
| `candidate` | Self | Default for sign-ups. |
| `recruiter` | Company-scoped | Belongs to one company. |
| `hiring_manager` | Company-scoped | Recruiter + can issue pledges. |
| `company_owner` | Company-scoped | Manages members and billing. |
| `admin` | Platform | Internal staff. |
| `moderator` | Platform | Reviews content; cannot adjust money. |
| `support` | Platform | Read access for help cases. |

## Permission Matrix (selected)

| Action | candidate | recruiter | hiring_mgr | owner | admin |
|---|---|---|---|---|---|
| Publish bounty | — | ✓ | ✓ | ✓ | ✓ |
| Issue recruiter pledge | — | — | ✓ | ✓ | — |
| Manage company members | — | — | — | ✓ | ✓ |
| Resolve dispute | — | — | — | — | ✓ |
| Adjust XP | — | — | — | — | ✓ |
| Read another user's KYC data | — | — | — | — | ✓ (audit-logged) |

## Multi-Tenancy
Single shared schema; tenant isolation via `company_id` checks at the service layer. Every bounty/pipeline query is scoped by `company_id`.

## Enforcement Points
- API gateway: coarse role check.
- Service layer: ABAC predicates.
- DB: row-level checks for company-scoped tables (Postgres RLS optional).

## Audit
Every privileged action (admin, moderator) writes to `audit_log`.

## Open Questions
- Postgres RLS now or later?
- Custom roles per enterprise customer or fixed set?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
