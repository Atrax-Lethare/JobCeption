# Infrastructure Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** DevOps
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 07-containerization-spec.md, 07-environments-spec.md, 07-scalability-spec.md

## Purpose
Cloud, regions, networking, and IaC layout.

## Cloud
- Primary: AWS.
- Region: `ap-south-1` (Mumbai) for India launch.
- Secondary region (DR): `ap-southeast-1` (Singapore) post-launch.

## Network Topology
- VPC with public and private subnets across 3 AZs.
- NAT gateway for egress.
- Internal services in private subnets only.
- ALB → API gateway → services.
- Sandbox subnet hardened with egress-only NAT and per-bounty allow-listing.

## Compute
- EKS for application services.
- Dedicated node group for sandbox workers (taints + tolerations).
- Spot for sandbox burst; on-demand for control plane.

## Data Stores
- RDS Postgres (Multi-AZ).
- ElastiCache Redis cluster.
- S3 for object storage with bucket versioning + object lock for legal data.
- Self-hosted Meilisearch on EKS or managed.

## Networking & Security
- WAF in front of ALB.
- Shield Standard (Advanced if budget allows).
- TLS termination at ALB; mTLS internally.

## DNS & Edge
- Route 53.
- CloudFront for static assets.

## IaC
- Terraform monorepo with per-environment workspaces.
- Module structure: `network/`, `eks/`, `data/`, `secrets/`, `dns/`.
- All changes via PR; `terraform plan` posted to PR.

## Open Questions
- AWS vs. GCP final call.
- Managed vs. self-hosted Postgres at scale.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
