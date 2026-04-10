# DevOps Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** DevOps
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** —

## Purpose
CI/CD, branching, release process, rollback.

## Source Control
- GitHub.
- Trunk-based development; short-lived feature branches.
- Conventional commits.

## CI Pipeline
1. Lint + format check.
2. Type check.
3. Unit tests.
4. Integration tests (against ephemeral DB).
5. Build container image.
6. Image scan (Trivy).
7. Push to ECR with commit SHA tag.
8. SAST (Semgrep / CodeQL).

## CD Pipeline
- Auto-deploy `main` to staging.
- Manual promotion to prod with canary (10% → 50% → 100%).
- Database migrations run as a separate job before app deploy.
- Rollback by re-deploying previous image tag.

## Release Cadence
- Continuous to staging.
- Prod releases up to multiple per day for non-pledge services.
- Pledge service releases gated behind extra review.

## Branching
- `main` is releasable.
- Feature branches off `main`.
- Hotfix branches off the last released tag.

## Versioning
- Semantic version on app + tag on git release.
- API has its own URI version (`/v1/`).

## Open Questions
- Blue/green for the pledge service vs. canary.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
