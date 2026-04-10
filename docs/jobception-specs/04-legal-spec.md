# Legal Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Legal
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 00-glossary.md, 09-trust-and-reputation-spec.md

## Purpose
Define the legal artifacts and the binding interpretation of platform mechanics like pledges, reviews, IP, and disputes.

## Documents
- Terms of Service.
- Privacy Policy.
- Cookie Policy.
- Acceptable Use Policy.
- Recruiter Agreement (additional).
- Bounty Participant Agreement.
- Escrow Terms.
- Dispute Resolution Policy.

## Pledges as Legal Artifacts
- A **Pledge** is a recorded statement of intent to enter (or to extend) an employment relationship under stated terms.
- It is not, by itself, an employment contract; it is **evidence** of the parties' commitments.
- Whether a Pledge constitutes a binding offer depends on jurisdiction. In India, a Pledge plus subsequent conduct may constitute a binding offer under the Indian Contract Act, 1872.
- Platform retains pledge records for 7 years (see `02-data-governance.md`).

## Reviews & Defamation
- Reviews are user-generated content; the platform is a host.
- Right of Reply is mandatory before review goes live (24 h window).
- Reviews can be challenged via Dispute; defamatory or verifiably false reviews are removed.
- Aggregate ratings are platform-derived statements of fact, not opinion.

## IP Ownership of Bounty Submissions
Default rules (configurable per bounty):
- **XP bounties:** candidate retains all IP. Platform receives a non-exclusive license to display the work on the candidate's profile.
- **Job bounties:** candidate retains IP unless explicitly assigned in the bounty terms; recruiter receives evaluation rights.
- **Freelance bounties:** IP transfers to the client on payment release; transfer terms are part of the bounty publish form.

## Acceptable Use
- No harassment, hate, illegal content, or scams.
- No automated submission farms.
- No off-platform payment for bounty rewards.
- Violations → strikes → suspension.

## Dispute Resolution Tiering
1. Direct: parties resolve via in-app messaging.
2. Mediated: platform Admin reviews evidence.
3. Arbitration: jurisdiction-specific binding arbitration clause in ToS.

## Open Questions
- Choice of governing law and forum.
- Whether to bind users to arbitration or allow class action.
- Jurisdiction-specific pledge enforceability mapping.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
