# Database Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Backend
- **Last Updated:** 2026-04-09
- **Depends On:** 02-domain-model.md, 02-data-governance.md
- **Referenced By:** —

## Purpose
Concrete database choices, schema policies, indexing strategy, migration workflow, backup/restore, and retention.

## Database Choice
- **Primary:** PostgreSQL 16 (ADR-0001).
- **Search:** Meilisearch or OpenSearch for bounty discovery.
- **Cache:** Redis 7.
- **Object storage:** S3-compatible for submissions, attachments, documents.
- **Time-series (optional):** TimescaleDB extension for analytics events.

## Schema Conventions
- One schema per bounded context: `identity`, `bounty`, `pipeline`, `reputation`, `connections`, `escrow`, `audit`.
- Cross-schema FKs allowed only for stable, append-only references.
- All tables include audit columns from the domain model.
- Enums implemented as VARCHAR + CHECK; reserved for stable values.

## Indexing Strategy
- B-tree on every FK.
- Partial indexes on `status` columns for hot paths (e.g. `bounty WHERE status = 'published'`).
- GIN on `tags` and `jsonb` columns used in filters.
- Composite indexes for marketplace queries: `(bounty_type, status, published_at DESC)`.
- Full-text via Meilisearch, NOT Postgres FTS, to keep DB load predictable.

## Migrations
- Tool: `prisma migrate` OR `sqlc + golang-migrate` (decide in ADR-0001).
- Forward-only migrations; no destructive `down` migrations in prod.
- Every migration reviewed by the data owner and Tech Lead.
- Migrations run as a separate CI job, never as part of app boot.

## Backup & Restore
- Automated daily snapshots, retained 35 days.
- Weekly full backups retained 12 months.
- Monthly cold archive retained 7 years (legal — pledges).
- Quarterly restore drill into a sandboxed env.

## Retention
- **Pledges:** 7 years (legal evidence).
- **Reputation events:** indefinite while account active; anonymized after deletion request (see `02-data-governance.md`).
- **Bounty submissions:** 18 months from completion, then archived to cold storage.
- **Sandbox session logs:** 90 days.
- **Audit log:** 3 years hot, then cold for 7.

## Multi-Tenancy
Not multi-tenant — single shared schema. Companies are rows, not tenants.

## Open Questions
- Sharding strategy if write volume outgrows a single primary.
- Hot/cold separation for old bounty submissions.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
