# Monetization Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Product / Finance
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** —

## Purpose
Pricing, billing, escrow flows, invoicing, and tax.

## Revenue Model (proposed)
- **Candidates:** free.
- **Recruiters / Companies:** subscription tiers.
  - **Starter:** small teams, low monthly volume.
  - **Growth:** mid-size, more bounties, more sandbox sessions, dedicated support.
  - **Enterprise:** custom, SSO, SLA.
- **Freelance bounties:** platform fee on the reward (escrow).
- **Badges:** in-platform purchase; revenue split with the platform.
- **Add-ons:** verified-priority listing, sponsored bounties, premium templates.

## Billing
- Stripe Billing for subscriptions.
- Indian GST handled via Razorpay or Stripe Tax.
- Annual discount available.

## Escrow Flow (Freelance)
1. Recruiter publishes bounty → funds debited and held in escrow.
2. Bounty completed → recruiter selects winner.
3. On winner selection → funds released to candidate, minus platform fee.
4. Disputes pause release; mediation determines split.

## Invoicing
- Auto-generated, downloadable from billing dashboard.
- GSTIN field for Indian companies.

## Refunds
- Subscription: prorated on downgrade.
- Escrow: full refund if bounty closed before any submission.

## Cost-of-Goods
- Each bounty has an associated COG (sandbox compute, AI calls).
- Tracked per company for unit economics.

## Open Questions
- Free tier for tiny companies?
- Per-hire commission as an alternative model?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
