# Containerization Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** DevOps
- **Last Updated:** 2026-04-09
- **Depends On:** 03-sandbox-execution-spec.md
- **Referenced By:** —

## Purpose
Image policy, build pipeline, registry, and runtime conventions.

## Image Standards
- Base: distroless or minimal Alpine.
- Multi-stage builds; final image contains only runtime artifacts.
- Pinned versions; no `:latest`.
- Non-root user; read-only filesystem where possible.
- Healthcheck declared.
- Labels: `org.opencontainers.image.*`.

## Registry
- Amazon ECR.
- Image scanning on push (Trivy + Inspector).
- Signed with Cosign; cluster admission controller verifies signatures.

## Sandbox Worker Images
- One per supported language/framework.
- Built nightly with latest patches.
- Stricter policy than app images: no shell, no package manager, no curl/wget at runtime.
- See `03-sandbox-execution-spec.md`.

## Runtime
- Kubernetes pods with resource requests + limits.
- PodSecurityStandard `restricted` enforced.
- NetworkPolicies default-deny.

## Open Questions
- Alpine vs. distroless across the board.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
