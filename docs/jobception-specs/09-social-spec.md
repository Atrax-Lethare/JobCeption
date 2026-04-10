# Social / Connections Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Product / Backend
- **Last Updated:** 2026-04-09
- **Depends On:** 02-domain-model.md, 09-trust-and-reputation-spec.md
- **Referenced By:** —

## Purpose
Define the Connections layer — posts, feed, badges, notifications, presence — and how it interacts with the reputation economy.

## Posts
- Markdown body, optional attachments (images, project links, video embeds).
- Length cap: 5,000 chars body.
- Edit window: 15 minutes after publish; edits flagged.
- Delete: soft delete with tombstone.

## Feed (For You)
- Personalized ranking by Visibility Algorithm.
- Inputs: follow graph, recent engagement, badges received, AI-slop score (negative weight), recency.
- Cold-start: domain interest pickers at signup.

## Visibility Algorithm (sketch)
```
score = w1*log(1+likes) + w2*log(1+comments) + w3*badge_value
      + w4*follower_quality(author) + w5*recency_decay
      - w6*ai_slop_score - w7*reports
```
Weights tunable via config. Documented and revisable; changes logged.

## Spam Folder
- Posts with `ai_slop_score > 0.8` quarantined automatically.
- Authors notified.
- Authors can appeal via Dispute.

## Badges
- Scarce: each user has a monthly allowance based on subscription tier.
- Paid: exceed the allowance via purchase.
- Each badge has a defined point value contributing to the post's visibility and the author's CAP.

## Notifications
- Channels: in-app, email, push (post-MVP).
- Categories: bounty (new round, results), pipeline (date set, document needed, pledge), social (likes, comments, badges, follows), system.
- Per-category prefs.
- Quiet hours respected.

## Messaging (post-MVP)
- 1:1 candidate ↔ recruiter, scoped to an active pipeline only.
- No DMs in marketplace pre-pipeline (avoids spam vectors).

## Presence
- Lightweight "active in last 24h" indicator. No real-time green dots — too creepy for hiring.

## Moderation
- Reports → admin queue.
- AUP violations → strikes.
- Coordinated brigading detection.

## Open Questions
- Comment threading depth.
- Whether to allow reposts.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
