# Glossary

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Product
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 02-domain-model.md, 03-api-spec.md, 04-legal-spec.md, 05-ui-ux-spec.md, 09-trust-and-reputation-spec.md

## Purpose
The canonical vocabulary of JobCeption. Every other spec — domain model, API, UI copy, legal documents — MUST use these terms exactly as defined here. If a term is missing or ambiguous, propose an addition via PR rather than inventing a synonym.

## Conventions
- **Bold** terms within definitions cross-reference other glossary entries.
- ⚠ marks anti-patterns JobCeption is designed to eliminate; they appear in industry context only.
- Where a term has precise legal meaning (e.g., **Pledge**), the binding interpretation lives in `04-legal-spec.md`.
- Acronyms are spelled out on first use within a spec, then abbreviated.

---

## A

### Admin
Internal JobCeption staff role with elevated permissions for moderation, dispute resolution, and platform configuration. Defined in `04-authorization-spec.md`.

### AI-Slop ⚠
Low-effort, AI-generated content that adds no human signal. The platform actively detects and quarantines AI-slop in **Posts** (via the **Spam Folder**) and flags it in **Bounty Submissions** (via **Anti-Cheat**).

### Anti-Cheat
The collection of mechanisms — tab tracking, paste detection, AI-output classifiers, plagiarism comparison, behavioral anomaly detection — that protect **Bounty** integrity. Configurable per **Bounty Template**. See `03-sandbox-execution-spec.md`.

### Appeal
A user's formal request to overturn a **Strike**, **Suspension**, or **Review**. Handled through the **Dispute** workflow.

### Applicant Tracking System (ATS) ⚠
The legacy resume-parsing software JobCeption is built to replace. Used here only as a reference point.

### Aura Points (AP)
A candidate's primary skill XP, earned through **Bounty** completion, **Endorsements**, and on-time **Pledge** fulfillment. AP is the headline number on the **Dynamic Profile**. Cannot be purchased. Formulas in `09-trust-and-reputation-spec.md`.

### Availability Status
A profile field with three values: `open_to_bounties`, `open_to_offers`, `not_looking`. Drives discovery filtering and notification eligibility. Respects **Incognito Mode**.

### Avatar Frame
A cosmetic border around a profile picture. Unlocked at AP milestones. Purely decorative; no gameplay effect.

---

## B

### Badge
A scarce, paid signal of appreciation that any user can award to a **Post**. Badges feed into **Creative Aura Points**. Cost and supply rules in `09-monetization-spec.md`.

### Binding Declaration
Any commitment submitted through the platform that, once made, cannot be retracted without reputational cost. Examples: publishing a **Bounty** (binds the recruiter to the **Salary Range** and JD), ticking the **Layer of Interest** "for the job", accepting a **Pledge Pop-up**.

### Black Hole ⚠
The industry pattern where an application is received and never acknowledged. JobCeption eliminates this — every **Bounty Submission** receives a structured **Performance Report**.

### Bounty
A scoped, time-boxed challenge published by a **Recruiter** that doubles as a job application, freelance gig, or pure-XP competition. The atomic unit of work on JobCeption. Every bounty has a **Bounty Type**, declared **Constraints**, one or more **Bounty Rounds**, and an evaluation method. Defined in `02-domain-model.md`.

### Bounty Round
A single stage within a **Bounty** (e.g., "screening", "deep dive", "live build"). Each round has its own deadline, rubric, and **Performance Report**.

### Bounty Submission
A candidate's deliverable for a single **Bounty Round** — code, design files, written response, recorded session, or sandbox state snapshot. Submissions are immutable once finalized.

### Bounty Template
A pre-built bounty structure. Two flavors: **Platform Templates** (curated by JobCeption) and **Creative Templates** (recruiter-customized, subject to platform review).

### Bounty Type
One of: `job` (full-time hiring), `freelance` (paid project), `xp` (skill practice, no hiring/payment), `competition` (cash prizes, often sponsored).

---

## C

### Candidate
A platform user whose primary intent is to find work, build skill, or earn rewards. Distinct from **Recruiter**. A single human can hold both a candidate and a recruiter identity but they are separate accounts.

### Candidate Reputation
The aggregate trust score for a candidate, derived from **Pledge Honor Rate**, on-time submission rate, **Review** aggregate, and **Strikes**. Distinct from **Aura Points**. See `09-trust-and-reputation-spec.md`.

### Company
A verified organizational entity that employs one or more **Recruiters**. Companies have their own profile page and **Company Rating**.

### Company Rating
A 0–5 score derived from **Pledge Honor Rate**, declared-vs-final salary delta, time-to-decision, feedback coverage, and aggregate candidate **Reviews**.

### Compliance Interview ⚠
The industry anti-pattern where a candidate is interviewed only to satisfy HR diversity or fairness policies after the role has effectively been filled internally. JobCeption's binding **Pledge** mechanism makes this visible and reviewable.

### Connections
The professional social layer of JobCeption — **Posts**, **Feed**, **Badges**, **Creative Aura Points**. Specified in `09-social-spec.md`.

### Constraints
Hard requirements a recruiter declares for a **Bounty** — minimum **Aura Points**, language/framework, time/space complexity, geography, etc. Constraints can be **Visible** (shown to candidates) or **Hidden** (silent filters).

### Counteroffer Flip ⚠
The industry pattern where a candidate verbally accepts an offer, then withdraws after their current employer makes a retention bid. The candidate-side **Pledge Pop-up** binds candidates to their acceptance.

### Creative Aura Points (CAP)
A parallel XP score earned through **Connections** activity (likes, badges, helpful posts). CAP is **Diluted** at a configurable ratio (default 10:1) before contributing to total XP, so social popularity cannot dominate hiring signal.

### Creative Template
A **Bounty Template** customized or built from scratch by a recruiter. Subject to platform review for fairness and anti-bias.

### CV (Auto-Generated)
A curriculum vitae built automatically from the candidate's on-platform work — completed **Bounties**, shared freelance projects, **Endorsements**. Manual additions are allowed but flagged as unverified.

---

## D

### Declaration of Intent
The candidate-side **Layer of Interest** tick-box: must select `for_job`, `for_xp`, or `for_reward` before joining a **Bounty**. Binding.

### Delay Flag
A visible marker on the **Progress Indicator** when either party misses an **SLA**. Delay flags are public on the company or candidate profile and feed into **Reputation**.

### Dilution Ratio
The factor by which **Creative Aura Points** are reduced before contributing to total displayed XP. Default 10:1; tunable by Admin.

### Dispute
A formal conflict between a candidate and a recruiter (or between either party and the platform) over a **Pledge**, **Review**, **Submission Evaluation**, or **Strike**. Resolved via the **Mediation** workflow.

### Document Request
A list of documents a recruiter requires from a candidate during the **Hiring Pipeline**, each with a deadline. Tracked on the **Progress Indicator**.

### Dynamic Profile
A candidate's living portfolio page — **Identity & Header**, **High-Level Stats**, **Posts**, **Bounties**, **CV**, **Skill Graph**, **Reputation Score**. The replacement for the static resume.

---

## E

### Endorsement
A peer attestation of a specific skill, capped per source to prevent farming. Contributes a small amount of **Aura Points**.

### Escrow
A held cash balance for **Freelance** and **Competition** bounty rewards. Funds are deposited at bounty publish time and released only when the bounty is awarded. Specified in `09-monetization-spec.md`.

---

## F

### Feed (For You)
The personalized **Connections** stream surfaced to a user, ranked by the **Visibility Algorithm**.

### Freelance Bounty
A **Bounty Type** with a fixed cash **Reward**, paid via **Escrow**. No employment relationship is created.

---

## G

### Ghost Rate ⚠
The percentage of hiring pipelines that end without an explicit decision from one side. A platform-wide success metric JobCeption targets near zero. Tracked in `09-analytics-spec.md`.

---

## H

### Hidden Constraint
A **Constraint** the recruiter has chosen not to reveal to candidates. Used for silent filtering.

### Hiring Pipeline
The sequence of stages between a successful **Bounty** outcome and a final hire — interviews, document submission, offer, pledge. Managed by the **Progress Indicator**.

### Hiring Manager
A specific role within a **Company** with permission to publish **Bounties**, run **Hiring Pipelines**, and issue **Pledges**. Distinct from a **Recruiter** in larger organizations; collapsed in small ones.

---

## I

### Incognito Mode
A profile setting that hides a candidate's activity from specified companies (typically their current employer). Searchable by allow-listed recruiters only.

---

## L

### Layer of Interest
The mandatory **Declaration of Intent** at bounty signup. Binding.

---

## M

### Mediation
The platform-managed workflow for resolving a **Dispute**. May involve human Admin review, automated evidence review, or both.

---

## P

### Performance Report
A structured per-round (and final) report delivered to both candidate and recruiter after a **Bounty Round**. Includes constraint satisfaction, ranking, strengths, and weaknesses. Eliminates the **Black Hole** and **Zero Feedback** anti-patterns.

### Platform Template
A **Bounty Template** curated and maintained by JobCeption staff. Vetted for fairness and difficulty calibration.

### Pledge
A **Binding Declaration** of intent to honor a hiring outcome. Two forms:
- **Recruiter Pledge** — the company commits to hire the candidate at the declared terms.
- **Candidate Pledge** — the candidate commits to accept the offer and start on the agreed date.
Pledges are timestamped, logged, and legally binding to the extent permitted by jurisdiction (see `04-legal-spec.md`). Pledge violation triggers **Reputation** loss.

### Pledge Honor Rate
The ratio of honored to total **Pledges** for a user or company. A primary input to **Reputation**.

### Pledge Pop-up
The UI surface — a modal — through which a **Pledge** is issued. Requires explicit confirmation. Cannot be dismissed silently.

### Post
A piece of user-published content in **Connections**. Has a **Visibility Score**, can receive **Badges**, and can be flagged into the **Spam Folder**.

### Progress Indicator
The shared, real-time hiring pipeline tracker visible to both candidate and recruiter. Includes **Progress Bar**, **Document Requests**, **Pledge Pop-ups**, and the **Review** entry point. Specified in `01-functional-requirements.md`.

### Purple Squirrel ⚠
The industry anti-pattern of demanding an impossible combination of skills/experience/credentials. JobCeption's bounty-first model surfaces real candidates by demonstrated skill rather than résumé keyword match.

---

## R

### Recruiter
A platform user whose primary intent is to hire. Belongs to exactly one **Company**.

### Reputation
The umbrella term for trust signals attached to a user or company. Composed of **Aura Points**, **Creative Aura Points**, **Pledge Honor Rate**, **Review** aggregate, **Strikes**, and **Verified Identity** status. Specified in `09-trust-and-reputation-spec.md`.

### Review
A structured rating + comment pair attached to a candidate (by recruiter) or company (by candidate) after a **Hiring Pipeline** event. Subject to **Right of Reply** and **Dispute**.

### Reward
The cash prize attached to a **Freelance** or **Competition** bounty. Held in **Escrow**.

### Right of Reply
Every **Review** subject may post a single public response visible alongside the original review.

---

## S

### Salary Range
The recruiter-declared compensation band for a **Job Bounty**. Required at publish time. Cannot be lowered after publish without reputation cost.

### Sandbox
The isolated, instrumented runtime in which a candidate completes a **Bounty Submission**. Provides language/tool support, **Anti-Cheat**, and submission capture. Specified in `03-sandbox-execution-spec.md`.

### Skill Graph
A visualization of a candidate's demonstrated skills, derived from **Bounty** performance — not self-reported.

### SLA (Service-Level Agreement)
A platform-enforced response-time obligation. Examples: recruiter must evaluate a submission within X days; candidate must accept a **Pledge Pop-up** within Y days. SLA misses produce **Delay Flags**.

### Spam Folder
The **Connections** quarantine for low-effort and AI-generated **Posts**.

### Strike
A reputation penalty issued by Admin or auto-issued by the platform for verified misconduct. Three strikes within a window may trigger **Suspension**.

### Submission Evaluation
The result of evaluating a **Bounty Submission**. Has two layers: the platform's **Evaluation Layer** (independent, drives **Aura Points**) and the recruiter's evaluation (drives hiring decisions only).

### Suspension
A temporary or permanent ban from posting bounties or applying to them.

---

## T

### Time-to-Decision
Median elapsed time from a recruiter's first review of a submission to a final hire/no-hire call. A **Company Rating** input.

### Time-to-Hire
Median elapsed time from **Bounty** publish to a pledged offer. A platform success metric.

---

## V

### Verified Identity
A **Trust** badge earned by completing identity verification (government ID + liveness check). Required for issuing **Pledges** above certain reputation thresholds.

### Visible Constraint
A **Constraint** the recruiter has chosen to display publicly to candidates browsing the bounty.

### Visibility Algorithm
The ranking function for **Connections** **Posts** in the **Feed**.

### Visibility Score
A per-post numeric score from the **Visibility Algorithm**. Inputs: likes, comments, follower quality, dwell time, **Badge** count, recency, and AI-slop probability.

---

## X

### XP
Generic term for experience points. Two specific flavors exist on the platform: **Aura Points** and **Creative Aura Points**.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
