# Environments Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** DevOps
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** —

## Purpose
Define environment tiers, parity, secrets, feature flags, and configuration.

## Environments
| Env | Purpose | Data | Access |
|---|---|---|---|
| `dev` | Per-engineer or shared | Synthetic | Engineering |
| `staging` | Integration + QA | Anonymized prod snapshot | Engineering + QA |
| `prod` | Live | Real | Strictly limited |

## Parity
Staging mirrors prod topology; differences are documented and minimized.

## Secrets
- AWS Secrets Manager per env.
- No secrets in env files committed to git.
- Local dev uses `.env.local` ignored by git; values pulled from vault.

## Configuration
- Static config in repo per env.
- Dynamic config (rate limits, feature flags) via a managed service (LaunchDarkly or self-hosted Unleash).

## Feature Flags
- All risky features gated.
- Flags have an owner and a removal date.
- Stale flags pruned in monthly review.

## Open Questions
- Self-hosted Unleash vs. paid LaunchDarkly.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
