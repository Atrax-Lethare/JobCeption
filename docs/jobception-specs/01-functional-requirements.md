# Functional Requirements

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Product
- **Last Updated:** 2026-04-10
- **Depends On:** 01-product-overview.md, 01-personas-and-journeys.md
- **Referenced By:** 02-domain-model.md, 03-api-spec.md, 05-ui-ux-spec.md, 08-testing-strategy.md

## Purpose
Enumerate every feature with user stories and acceptance criteria. The contract between Product and Engineering.

## Requirement Conventions
- IDs follow `FR-<area>-<number>`.
- Each requirement has: user story, acceptance criteria, priority (P0/P1/P2), and dependent specs.

## FR-AUTH — Identity & Access
### FR-AUTH-001 — Sign up as candidate or recruiter (P0)
**Story:** As a new user, I can choose my identity at signup so the platform shows me the right interface.
**Acceptance:**
- Two clearly labelled signup paths.
- Email + password OR passkey.
- Recruiters must associate with a verified Company before publishing bounties.

### FR-AUTH-002 — Verified Identity (P0)
Government-ID + liveness check; visible badge on profile. Required to issue Pledges.

## FR-BOUNTY — Bounty Lifecycle
### FR-BOUNTY-001 — Recruiter Layer-1 Binding Form (P0)
- Required fields: title, type, JD, requirements, salary range (if job) OR reward (if freelance), location, employment type, start date.
- Once published, salary range and JD are immutable. Edits require explicit "amendment" with reputation impact.

### FR-BOUNTY-002 — Bounty Templates (P0)
- Recruiter chooses Platform Template or Creative Template.
- Creative Templates pass through fairness review before publish.

### FR-BOUNTY-003 — Constraints (P0)
- Recruiter declares min XP, language, tools, complexity bounds, geography.
- Each constraint flagged Visible or Hidden.

### FR-BOUNTY-004 — Candidate Discovery & Filter (P0)
- Marketplace page with filters by domain, type, reward/salary, location, XP threshold, company rating.

### FR-BOUNTY-005 — Layer of Interest (P0)
- Mandatory tick before joining a job bounty: `for_job` / `for_xp` / `for_reward`.
- Skipped for pure-XP bounties.

### FR-BOUNTY-006 — Sandbox Submission (P0)
- Candidate completes bounty inside the sandbox; submission auto-saved every N seconds.
- Submission immutable on finalize.

### FR-BOUNTY-007 — Performance Report (P0)
- Both candidate and recruiter receive structured per-round and final reports within SLA.
- Includes constraint satisfaction, ranking, strengths, weaknesses.

### FR-BOUNTY-008 — Project Sharing (P1)
- For freelance, candidate may opt to publish completed project to Dynamic Profile.

## FR-PIPE — Hiring Pipeline / Progress Indicator
### FR-PIPE-001 — Progress Bar (P0)
- Shared timeline visible to both parties; round dates editable by recruiter only; edits logged.

### FR-PIPE-002 — Document Requests (P0)
- Recruiter posts list with deadlines; candidate uploads with status.

### FR-PIPE-003 — Recruiter Pledge Pop-up (P0)
- Modal with explicit confirm; logs Pledge.

### FR-PIPE-004 — Candidate Pledge Pop-up (P0)
- Modal with explicit confirm; logs Pledge; binds candidate.

### FR-PIPE-005 — Mutual Reviews (P0)
- Either party may submit a Review at defined points; subject has Right of Reply.

### FR-PIPE-006 — SLA Delay Flags (P0)
- System auto-flags missed SLAs; flags display on profiles.

## FR-CONN — Connections
### FR-CONN-001 — Posts (P1)
- Users publish text, image, link, or attached project.

### FR-CONN-002 — For You Feed (P1)
- Personalized feed ranked by Visibility Algorithm.

### FR-CONN-003 — Badges (P1)
- Users buy and award scarce badges to posts.

### FR-CONN-004 — Spam Folder (P1)
- AI-slop and low-effort posts quarantined; users can review their spam folder.

### FR-CONN-005 — Creative Aura Points (P1)
- CAP earned from engagement; diluted into total XP.

## FR-PROFILE — Dynamic Profile
### FR-PROFILE-001 — Profile Page (P0)
- Header (avatar frame, username, tagline).
- Stats (Bounties, Followers, AP, CAP, Reputation Score).
- Tabs: Posts, Bounties, CV, Reviews.

### FR-PROFILE-002 — Auto-Generated CV (P0)
- Built from completed bounties and shared projects.

### FR-PROFILE-003 — Skill Graph (P1)
- Visualization derived from bounty results.

### FR-PROFILE-004 — Incognito Mode (P1)
- Hide activity from specified companies.

## FR-REP — Reputation & Trust
### FR-REP-001 — Aura Points System (P0)
- Earned from bounties, endorsements, on-time pledges. Specified in `09-trust-and-reputation-spec.md`.

### FR-REP-002 — Pledge Honor Tracking (P0)
- All pledges logged; honor rate visible.

### FR-REP-003 — Company Rating (P0)
- Aggregate score on every company profile.

### FR-REP-004 — Disputes & Appeals (P0)
- File, review, resolve, appeal.

## FR-MONEY — Monetization & Escrow
### FR-MONEY-001 — Freelance Escrow (P0)
- Funds held at publish; released on award.

### FR-MONEY-002 — Recruiter Plans (P1)
- Subscription tiers gating bounty publishing volume.

## FR-EVAL — Evaluation
### FR-EVAL-001 — Per-Type Evaluation Pipelines (P0)
- Code, design, writing, and data science bounties each have a dedicated evaluation pipeline.
- See `03-evaluation-spec.md`.

### FR-EVAL-002 — Hidden Test Cases (P0)
- Test cases stored server-side, never exposed to candidates.
- Sample / hidden / validation visibility tiers.

### FR-EVAL-003 — Performance Reports Within SLA (P0)
- Code/writing/DS: ≤ 15 min after submission close.
- Design (peer review): ≤ 72 h after review window close.

### FR-EVAL-004 — Two Independent Evaluations (P0)
- Platform evaluation (deterministic, drives AP) and Recruiter evaluation (subjective, drives hiring) never influence each other.

## FR-RECO — Recommendation & Search
### FR-RECO-001 — Personalized Bounty Marketplace (P1)
- Each candidate sees a ranked feed via the hybrid recommender (LightFM + sentence-transformers).

### FR-RECO-002 — Recruiter Semantic Search (P1)
- Natural-language candidate search via Weaviate + cross-encoder reranking.
- Hard filters layered after AI scoring.

### FR-RECO-003 — JD ↔ Candidate Match Score (P1)
- Single 0–100 score shown to recruiters on submissions screen and (optionally) candidates on bounty detail.

### FR-RECO-004 — Match-Triggered Notifications (P1)
- New bounty published → recommendation engine identifies high-match candidates → notification (rate-limited per candidate).

## FR-FRAUD — Fraud & Integrity
### FR-FRAUD-001 — Code Plagiarism Detection (P0)
- MOSS + MinHash/LSH on every code submission.
- Flag-and-queue, never auto-disqualify.

### FR-FRAUD-002 — AI-Generated Content Detection (P0)
- RoBERTa detector on code and writing submissions; signal in performance report.

### FR-FRAUD-003 — Behavioral Biometrics (P1)
- Keystroke dynamics + paste event tracking during sandbox sessions.
- Stored as anti-cheat flags; admin-visible.

### FR-FRAUD-004 — Review Brigading Detection (P1)
- Velocity + network graph checks; sudden negative review bursts auto-held for admin review.

### FR-FRAUD-005 — KYC Gating for High-Value Actions (P0)
- Identity verification required for issuing Pledges and for freelance bounties above a threshold.

## FR-ADMIN — Admin & Moderation
### FR-ADMIN-001 — Moderation Queue (P0)
### FR-ADMIN-002 — Dispute Resolution UI (P0)
### FR-ADMIN-003 — Reputation Adjustment Tools (P0)

## Open Questions
- Should bounty submissions be peer-reviewable as a fallback when recruiters miss SLAs?
- Are team bounties P1 or P2?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Added FR-EVAL, FR-RECO, FR-FRAUD requirement sections.
