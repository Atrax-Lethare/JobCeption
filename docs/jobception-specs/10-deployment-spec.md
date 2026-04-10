# Deployment Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** DevOps
- **Last Updated:** 2026-04-09
- **Depends On:** 07-devops-spec.md, 07-environments-spec.md
- **Referenced By:** —

## Purpose
Define deploy targets, deploy strategies, rollback, and the special handling for the Pledge service.

## Targets
- EKS clusters per environment.
- Static frontend → CloudFront via S3.

## Strategies
- **Default services:** rolling update with maxSurge=25%, maxUnavailable=0.
- **Pledge service, Escrow service:** blue/green to eliminate any window of mixed-version writes.
- **Sandbox workers:** image refresh via pool drain — old workers complete current sessions; new sessions go to new workers.

## Canary
- 10% → 50% → 100% with 10-minute soak windows.
- Auto-abort on error rate spike or latency regression.

## Database Migrations
- Run as a separate Job before app rollout.
- Backwards compatible by policy (expand → migrate → contract).
- A migration that requires downtime requires a maintenance window and explicit approval.

## Rollback
- Re-deploy previous image tag (kept for 30 days).
- DB rollback only via forward-fix (no down migrations).

## Maintenance Windows
- Avoided when possible.
- Communicated 72 h in advance via in-app banner and email.

## Open Questions
- Do we need read-only mode for the platform during a maintenance window?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
