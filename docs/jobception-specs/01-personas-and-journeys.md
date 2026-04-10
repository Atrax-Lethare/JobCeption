# Personas & Journeys

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Product / Design
- **Last Updated:** 2026-04-09
- **Depends On:** 01-product-overview.md
- **Referenced By:** 05-ui-ux-spec.md

## Purpose
Define the people JobCeption serves, their goals, and the end-to-end journeys they take through the product. Used as the source of truth for UX flows and acceptance criteria.

## Personas

### P1 — "Riya the Early-Career Candidate"
- **Background:** 2 years experience, frontend developer at a mid-sized company in Bengaluru.
- **Goals:** Move to a better-paying role; prove she can do more than her current title suggests.
- **Pains:** ATS rejections, take-home tests that get ghosted, salary opacity.
- **Wins on JobCeption when:** She completes a bounty, gets a structured performance report, and lands a role with a published salary range she trusts.

### P2 — "Arjun the Career Switcher"
- **Background:** 6 years in mechanical engineering, self-taught data analyst, no formal CS credentials.
- **Goals:** Be evaluated on what he can do today, not on his degree.
- **Pains:** Years-of-experience filters, irrelevant credential checks.
- **Wins when:** He builds an XP base through pure-XP bounties, then converts that XP into job bounty access.

### P3 — "Meera the Senior Practitioner"
- **Background:** 12 years experience, staff designer, currently employed.
- **Goals:** Quietly explore opportunities without alerting her current employer.
- **Pains:** Public job-board profiles, recruiter spam, slow processes.
- **Wins when:** She uses Incognito Mode to selectively engage with high-rated companies.

### P4 — "Karan the In-House Recruiter"
- **Background:** Recruiter at a 200-person Series B startup.
- **Goals:** Make 8 quality hires per quarter without drowning in spam.
- **Pains:** 500 irrelevant applications per role; hiring manager bottlenecks.
- **Wins when:** Pre-filtered, demonstrably skilled candidates flow into a transparent pipeline.

### P5 — "Priya the Freelance Client"
- **Background:** Founder of a small agency, occasionally needs short project work.
- **Goals:** Get a real working deliverable from a vetted freelancer with predictable cost.
- **Pains:** Unreliable freelancers, payment disputes, cold-start trust.
- **Wins when:** She posts a freelance bounty with escrowed payment and gets working code in a week.

### P6 — "JobCeption Admin"
- **Background:** Internal staff, trust & safety / dispute resolution.
- **Goals:** Keep the platform fair and the reputation system uncorrupted.
- **Pains:** Brigading, fake reviews, dispute backlogs.

## Jobs to Be Done (JTBD)

- **Candidate JTBD:** "When I'm looking for work, I want my actual ability to be visible and rewarded, so that I can land roles based on what I can do, not who I know."
- **Recruiter JTBD:** "When I'm filling a role, I want pre-filtered, accountable candidates so I can hire faster without sacrificing quality."
- **Freelance Client JTBD:** "When I have a one-off project, I want vetted talent and protected payment so the work actually ships."

## Key Journeys

### J1 — Candidate's First Bounty
1. Sign up → verify email → build skeleton **Dynamic Profile**.
2. Browse the **Bounty** marketplace; filter by domain and reward.
3. Tick the **Layer of Interest**.
4. Enter the **Sandbox**, complete the bounty within the deadline.
5. Submit → receive automated **Performance Report** within the SLA.
6. Earn **Aura Points** that appear on the profile.

### J2 — Job Bounty to Hire
1. Recruiter publishes bounty (Layer-1 binding form, Layer-2 template, Layer-3 constraints).
2. Candidates apply via Sandbox; Layer-4 performance reports go to recruiter.
3. Recruiter advances top candidates into the **Hiring Pipeline**.
4. **Progress Indicator** drives time-boxed rounds with **Document Requests**.
5. **Recruiter Pledge Pop-up** issues the offer.
6. **Candidate Pledge Pop-up** binds acceptance.
7. Both sides earn / lose reputation based on pledge fulfillment.

### J3 — Freelance Bounty
1. Client publishes freelance bounty with cash reward; funds escrowed.
2. Candidate completes; client evaluates; reward released from escrow.
3. Candidate optionally publishes the project to their profile.

### J4 — Dispute
1. One party files a dispute citing pledge violation, unfair review, or evaluation error.
2. Mediation queue picks up the case within SLA.
3. Evidence review (timestamps, sandbox logs, message history).
4. Resolution: uphold, partial, or overturn. Reputation updated. Right of appeal preserved.

## Open Questions
- Do we need a "Hiring Manager" persona distinct from "Recruiter"?
- How do we onboard the first cohort of candidates before any companies are present (cold start)?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
