# API Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Backend
- **Last Updated:** 2026-04-09
- **Depends On:** 02-domain-model.md, 04-authorization-spec.md
- **Referenced By:** —

## Purpose
Define API style, conventions, versioning, error handling, and the endpoint surface.

## Style
- RESTful JSON over HTTPS.
- Resource-oriented URLs: `/v1/bounties/{id}/rounds/{round_id}/submissions`.
- Plural nouns; lowercase; kebab-case for multi-word.

## Versioning
- URI versioning: `/v1/`.
- Breaking changes only in a new major; minimum 6-month deprecation window.
- API specs published as OpenAPI 3.1; generated from code annotations.

## Authentication
- Bearer JWT in `Authorization` header.
- Tokens issued by Identity Service; short-lived (15 min) + refresh tokens.
- Service-to-service: mTLS.

## Pagination
- Cursor-based by default: `?cursor=&limit=`.
- Max `limit` 100; default 20.
- Response: `{ data: [...], next_cursor: "..." }`.

## Filtering & Sorting
- Query params: `?filter[bounty_type]=job&sort=-published_at`.
- Sort prefix: `-` for descending.

## Idempotency
- All write endpoints accept `Idempotency-Key` header (required for pledges and escrow).
- Server stores key hash + response for 24 h.

## Errors
Single envelope:
```json
{
  "error": {
    "code": "BOUNTY_ALREADY_PUBLISHED",
    "message": "Salary range cannot be changed after publish.",
    "details": {},
    "request_id": "..."
  }
}
```
- HTTP status reflects category; `code` is the canonical machine identifier.
- Codebook maintained alongside this spec.

## Rate Limiting
- Default: 600 req/min per user, 60 burst.
- Bounty submission: 1 finalize per round per participation.
- Pledge endpoints: 10/min per user.

## Endpoint Surface (selected)
### Identity
- `POST /v1/auth/signup`
- `POST /v1/auth/login`
- `POST /v1/auth/refresh`
- `GET /v1/me`

### Bounties
- `POST /v1/bounties` (recruiter)
- `GET /v1/bounties` (marketplace, public)
- `GET /v1/bounties/{id}`
- `POST /v1/bounties/{id}/publish`
- `POST /v1/bounties/{id}/participations` (candidate joins)
- `POST /v1/participations/{id}/withdraw`

### Sandbox
- `POST /v1/sandbox/sessions` → returns websocket URL
- `POST /v1/sandbox/sessions/{id}/finalize`

### Pipeline & Pledges
- `POST /v1/pipelines/{id}/stages`
- `POST /v1/pipelines/{id}/document-requests`
- `POST /v1/pipelines/{id}/pledges` (kind: recruiter_offer | candidate_acceptance) — **idempotency key required**

### Reviews
- `POST /v1/reviews`
- `POST /v1/reviews/{id}/reply`

### Connections
- `POST /v1/posts`
- `GET /v1/feed`
- `POST /v1/posts/{id}/badges`

### Disputes
- `POST /v1/disputes`
- `GET /v1/disputes/{id}`

## Webhooks
Outbound webhooks for: pledge issued, pledge honored/violated, escrow released. HMAC-signed.

## Open Questions
- REST vs. GraphQL for the candidate profile screen (lots of joins).
- WebSocket-only vs. SSE for sandbox event stream.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
