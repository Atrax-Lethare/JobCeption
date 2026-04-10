# Error Handling Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Frontend
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** —

## Purpose
Define how the client handles errors — display, retry, fallback, and reporting.

## Categories
- **Network errors** (offline, timeout): inline retry + offline banner.
- **Validation errors** (4xx with field details): per-field inline messages.
- **Auth errors** (401 / 403): redirect to login / show "permission denied" page.
- **Conflict errors** (409): show server message with refresh CTA.
- **Server errors** (5xx): generic error UI + retry; Sentry capture.
- **Rate limit** (429): friendly "too fast" message with retry-after countdown.
- **Sandbox-specific**: separate error surface; never block submission save.

## Retry Policy
- Reads: 3 attempts with exponential backoff (200 ms, 600 ms, 1.5 s).
- Writes (non-idempotent): no automatic retry; explicit user action.
- Writes (idempotent, with key): up to 3 attempts.

## Error Boundaries
- Per route group + per critical component (sandbox editor, pledge dialog).
- Boundary captures, reports to Sentry, renders fallback UI with reload CTA.

## Reporting
- Sentry with PII scrubbing.
- Distinguish handled vs. unhandled errors.
- Tag with route, user role, build hash.

## User Messaging
- No raw stack traces.
- Always actionable: what happened + what they can do.
- Pledge / escrow errors get extra-careful copy and a support deeplink.

## Open Questions
- Acceptable error rate before showing platform-wide "we're investigating" banner.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
