# Authentication Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Security / Backend
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** —

## Purpose
Define authentication flows, session management, MFA, and passkey support.

## Identity Providers
- Email + password.
- Passkeys (WebAuthn).
- OAuth SSO: Google, GitHub, LinkedIn (post-MVP).
- Recruiter SSO via SAML/OIDC for enterprise plans.

## Password Policy
- Minimum 12 characters.
- Argon2id hashing.
- Breach check via HaveIBeenPwned k-anonymity API.

## MFA
- TOTP (default).
- WebAuthn second factor.
- SMS OTP only as recovery, not primary.

## Sessions
- JWT access tokens (15 min) + refresh tokens (30 days, rotated).
- Refresh tokens stored httpOnly + Secure cookie.
- Device list visible in user settings; revocable.

## Account Recovery
- Email reset link (15 min validity).
- Recovery codes generated at MFA setup.

## Verified Identity (KYC)
- Provider: see `03-integration-spec.md`.
- Required for: issuing pledges above $X reward, recruiters publishing bounties.
- Stored in C3 classification; image deleted after verification, only pass/fail kept.

## Sign-up Flow
1. Email + password OR passkey.
2. Email verification.
3. Choose identity (candidate / recruiter).
4. Profile basics.
5. Optional: KYC (mandatory for recruiters before publish).

## Open Questions
- Mandatory MFA for all users or only recruiters?
- Allow social SSO at MVP?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
