# Security Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Security
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 03-sandbox-execution-spec.md, 04-auth-spec.md, 04-authorization-spec.md, 07-infrastructure-spec.md

## Purpose
Threat model, encryption policy, secrets handling, vulnerability management, and incident response baseline.

## Threat Model (STRIDE summary)
- **Spoofing:** account takeover, impersonation. Mitigated by MFA, passkeys, session monitoring.
- **Tampering:** modifying pledges or reputation. Mitigated by immutable storage, audit logs, DB triggers.
- **Repudiation:** "I never pledged that". Mitigated by signed evidence (IP, UA, timestamp) on every pledge.
- **Information disclosure:** leaking salary or PII. Mitigated by RBAC, encryption, classification.
- **Denial of service:** sandbox flood, search hammering. Mitigated by rate limits, queues, autoscaling.
- **Elevation of privilege:** sandbox escape, admin role abuse. Mitigated by isolation (`03-sandbox-execution-spec.md`) and admin audit log.

## Encryption
- **In transit:** TLS 1.3 only; HSTS; cert pinning for mobile.
- **At rest:** AES-256 via cloud KMS for DB, object storage, backups.
- **Application-level:** PII columns (KYC) encrypted with envelope encryption; key rotation annual.

## Secrets Management
- AWS Secrets Manager.
- Zero plaintext secrets in code, env files, logs, or error messages.
- Short-lived credentials via IAM roles wherever possible.
- Quarterly secret rotation.

## Vulnerability Management
- Dependency scanning in CI (Snyk / Dependabot).
- Container image scanning (Trivy).
- SAST in CI (Semgrep / CodeQL).
- DAST against staging weekly.
- External penetration test annually.

## Incident Response
- On-call rotation.
- Severity ladder: SEV1 (data breach, full outage) → SEV4 (cosmetic).
- Postmortem within 5 business days for SEV1/SEV2; blameless format.

## Logging & Monitoring
- All authn events, authz failures, pledges, admin actions in audit log.
- Anomaly alerts on: bulk reputation changes, mass signup, unusual sandbox activity.

## Bug Bounty
- Public program once platform stabilizes.

## Open Questions
- Minimum acceptable security maturity at MVP launch.
- Pen test before or after public launch.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
