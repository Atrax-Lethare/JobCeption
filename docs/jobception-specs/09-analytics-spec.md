# Analytics Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Product / Data
- **Last Updated:** 2026-04-09
- **Depends On:** 02-domain-model.md, 02-data-governance.md
- **Referenced By:** —

## Purpose
Define the events we collect, how funnels are constructed, and the tooling.

## Tooling
- **Event collection:** PostHog (self-hosted) or Segment.
- **Warehouse:** BigQuery or Snowflake (post-MVP).
- **Dashboards:** Grafana for ops; Metabase for product.

## Event Conventions
- snake_case names: `bounty_published`, `submission_finalized`, `pledge_issued`.
- Versioned schema; events with shape changes get a new name.
- Every event includes: `user_id` (if known), `session_id`, `ts`, `source`, `version`.

## Core Events
| Event | Trigger | Properties |
|---|---|---|
| `signup_started` | User clicks signup | `kind` |
| `signup_completed` | User verifies email | `kind` |
| `bounty_viewed` | Marketplace card click | `bounty_id`, `bounty_type` |
| `bounty_joined` | Layer of Interest submitted | `bounty_id`, `loi` |
| `submission_started` | Sandbox session opened | `bounty_id` |
| `submission_finalized` | Submit clicked | `bounty_id`, `duration_s`, `flags` |
| `pledge_issued` | Pledge confirmed | `pledge_kind`, `pipeline_id` |
| `pledge_outcome` | Pledge resolved | `outcome`, `pipeline_id` |
| `review_published` | Review submitted | `subject_kind`, `rating` |
| `dispute_filed` | Dispute submitted | `subject_kind` |
| `post_published` | Post submitted | `length`, `attachments` |
| `badge_awarded` | Badge purchased + given | `badge_kind`, `amount` |

## Funnels
1. **Candidate activation:** signup → first bounty join → first submission finalized.
2. **Recruiter activation:** signup → company verified → first bounty published → first hire pledged.
3. **Pledge → hire:** recruiter pledge → candidate pledge → start date.

## Retention Cohorts
- Weekly active candidates by signup cohort.
- Recruiter retention by company size.

## North-Star Metrics
- Pledged hires per week.
- Pledge honor rate.
- Ghost rate.

## PII
- No PII in event properties.
- User ID is a hashed surrogate, not the email.

## Open Questions
- Buy vs. self-host PostHog.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
