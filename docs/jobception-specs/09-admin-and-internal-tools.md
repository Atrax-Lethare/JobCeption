# Admin & Internal Tools

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Ops
- **Last Updated:** 2026-04-09
- **Depends On:** 04-authorization-spec.md, 09-trust-and-reputation-spec.md
- **Referenced By:** —

## Purpose
Define the internal tooling Ops needs to run JobCeption — moderation, dispute resolution, support, reputation adjustment, content removal.

## Tools

### Moderation Queue
- Lists reported content (posts, comments, profiles, bounties).
- Severity ranking from AI triage.
- Actions: dismiss, warn, remove, strike, suspend.
- Every action logged.

### Dispute Resolution Console
- Pulls all evidence for a dispute: pledge details, message history, sandbox events, reviews.
- Templated decisions with reason codes.
- Outcome triggers reputation adjustment automatically.
- Right of appeal preserved with single re-review.

### Reputation Adjustment Tool
- Manual XP/CAP correction.
- Mandatory reason field; mandatory ticket reference.
- All adjustments visible in user audit log.

### Support Console
- Read-only view of user account, with PII access gated and audit-logged.
- Ability to impersonate (read-only) with a 30-min token and explicit user consent.

### Content Removal Tool
- Soft-delete with reason.
- Notification sent to author.
- DMCA / legal removal flow.

### Bounty Audit
- View all edits and amendments to a bounty.
- Compare published-state to current-state.

## Access
- Tools live behind staff-only auth (separate from main app auth).
- Hardware-key MFA mandatory for admin actions.

## Open Questions
- Do we need a tier of "moderator" who can act on content but not money/reputation?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
