# UI/UX Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Design
- **Last Updated:** 2026-04-09
- **Depends On:** 05-design-system.md, 01-personas-and-journeys.md
- **Referenced By:** —

## Purpose
Define the screens, flows, states, interactions, and microcopy for every primary surface.

## Surface Inventory

### Public
- Marketing landing
- Bounty marketplace (public, paginated)
- Public bounty detail
- Public company profile
- Public candidate profile (if profile is public)

### Candidate App
- Onboarding (signup → identity → profile basics → first bounty CTA)
- Dashboard (in-progress bounties, suggested bounties, notifications)
- Bounty browser
- Bounty detail (with Layer of Interest tickbox)
- Sandbox screen
- Submission summary
- Pipeline tracker (Progress Indicator candidate-side)
- Pledge Pop-up
- Connections feed
- Post composer
- Profile (own + edit modes)
- Settings (account, notifications, privacy, incognito)

### Recruiter App
- Onboarding (signup → company verify → KYC → first bounty wizard)
- Recruiter dashboard
- Bounty composer (Layer 1 → 2 → 3 wizard)
- Bounty detail (recruiter view, with submissions)
- Submission review
- Pipeline manager
- Pledge Pop-up (recruiter side)
- Company profile editor
- Team management
- Billing & escrow

### Admin
- Moderation queue
- Dispute resolution
- Reputation adjustment
- Content removal
- User suspension

## Critical Interaction Patterns

### Layer 1 Binding Form
- Stepper with explicit "binding declaration" warnings before publish.
- Final review screen shows every immutable field highlighted.
- Confirm checkbox: "I understand these terms cannot be changed after publish."

### Pledge Pop-up
- Modal occupies center; cannot be dismissed by overlay click.
- Two confirmation steps: review terms → type "I PLEDGE" → click confirm.
- Records IP, UA, timestamp at confirm.
- After confirm: success state with shareable receipt.

### Sandbox Screen
- Three regions: file tree, editor, output panel.
- Persistent timer banner at top.
- Auto-save indicator.
- Submit button red until "ready to submit" toggle.

### Performance Report
- Score ring at top (0–100).
- Constraint satisfaction checklist.
- Strengths / weaknesses written feedback.
- Comparison to median (anonymized).
- Next steps for the candidate.

## States
Every screen documents: empty / loading / partial / error / success / forbidden.

## Microcopy Principles
- Warm but precise. Never cute about consequences.
- Pledge pop-up text reviewed by Legal.
- Errors say what happened AND what to do.

## Open Questions
- Do we need a guided tour for first-time recruiters?
- How explicit should we be about Anti-Cheat presence to candidates?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
