# Trust & Reputation Spec

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Product / Tech Lead
- **Last Updated:** 2026-04-10
- **Depends On:** 02-domain-model.md, 04-legal-spec.md
- **Referenced By:** 09-monetization-spec.md, 09-admin-and-internal-tools.md, 09-analytics-spec.md

## Purpose
Define how trust is built and lost on JobCeption — the XP formulas, pledge mechanics, reputation rules, dilution ratios, dispute flows, and how the platform resists gaming. This is the actual moat of the product.

## Principles
1. **Append-only history.** Reputation is built from a ledger of events, not from a mutable score column.
2. **Two scores, one display.** Aura Points (skill) and Creative Aura Points (social) are computed separately and combined with dilution.
3. **Asymmetric punishment.** Earning XP is steady; losing it is sharp. Bad behavior costs more than good behavior earns in the same window.
4. **Transparent rules.** The full formula and weights are public. No black-box ranking.
5. **Reversible by appeal, not by edit.** Reputation events can be reversed only via a Dispute resolution; the original event is never deleted.
6. **Anti-farming.** Diminishing returns within a window, peer caps, identity requirements.

---

## Aura Points (AP) — Skill XP

### Earning
| Event | Base AP | Notes |
|---|---|---|
| Bounty submission finalized | 5 | Floor reward for completing |
| Bounty round passed | 10–50 | Scaled by recruiter difficulty rating |
| Bounty won (top 1) | 100–500 | Scaled by bounty difficulty + competitor count |
| Top 10% finish | 50–250 | |
| Endorsement received | 2 | Capped at 5 endorsements per peer per quarter |
| On-time pledge honored (recruiter or candidate) | 25 | |
| Verified Identity completed | 50 | One-time |
| Dispute won as defender | 25 | |

### Difficulty Scaling
Difficulty `d ∈ [1.0, 5.0]` is set by the Platform Evaluation Layer based on:
- Median completion time.
- Failure rate.
- Constraint complexity.
- Historical recruiter rating.

`AP_earned = base × d`.

### Diminishing Returns
- Endorsement curve: AP from endorsements caps at 100/quarter to prevent farming.
- Bounty XP from any single template caps after 3 wins (templates get repetitive — cap incentivizes range).

### Decay
- AP does not decay. A bounty done in 2026 still counts in 2030.
- Visibility filter on profiles can show "AP earned in last 12 months" alongside lifetime.

---

## Creative Aura Points (CAP) — Social XP

### Earning
| Event | Base CAP |
|---|---|
| Post receives a like | 1 |
| Post receives a comment from a non-follower | 3 |
| Post receives a Bronze badge | 10 |
| Post receives a Silver badge | 30 |
| Post receives a Gold badge | 100 |
| Followed by a high-AP user (top 10%) | 5 |

### Dilution
CAP contributes to display XP at a configurable ratio (default **10:1**, i.e. 100 CAP → 10 display XP). Dilution ratio is set in platform config and changes are logged via ADR.

**Total Display XP** = `AP + (CAP / dilution_ratio)`

CAP cannot exceed AP in display weight: even if CAP/ratio > AP, display contribution is capped at AP.

### Anti-Slop
Posts auto-flagged as AI-slop earn zero CAP. Authors can dispute the flag.

---

## Reputation Score (0–100)
A separate trust score that captures behavior, not skill.

### Inputs
- **Pledge Honor Rate** (40%): rolling 24-month window of all pledges issued or accepted.
- **On-Time Submission Rate** (15%): bounty rounds and document requests.
- **Review Aggregate** (20%): mean of received review scores, weighted by reviewer reputation.
- **Strike Penalty** (15%): −20 per active strike, decays over 12 months.
- **Verified Identity** (10%): binary 0 or full credit.

Score is recomputed nightly and on each relevant event.

### Display
- Numeric score 0–100.
- Tier label: Bronze (<50), Silver (50–74), Gold (75–89), Platinum (≥90).
- Visible on every profile.

---

## Pledge Mechanics

### Pledge Issuance
- Issued via the Pledge Pop-up — explicit, non-dismissable, two-step confirm.
- Recorded with snapshot of all material terms (salary, start date, role title, location).
- Recorded with evidentiary metadata (IP, UA, timestamp, user ID).
- Stored immutably; only `outcome` is later updated.

### Pledge Honoring
- A pledge is `honored` if the corresponding hire completes (recruiter side) or the candidate joins on the agreed date (candidate side).
- A pledge is `withdrawn_with_cause` if a documented exception applies (e.g., failed background check, force majeure).
- A pledge is `withdrawn_without_cause` otherwise.

### Reputation Impact
| Outcome | AP Δ | Reputation Δ |
|---|---|---|
| Honored | +25 | +2 |
| Withdrawn with cause | 0 | 0 |
| Withdrawn without cause (first time) | −100 | −10 |
| Withdrawn without cause (repeat) | −250 | −25 |

Repeat = a second within a 12-month window.

### Cause Catalogue
- Failed legal background check (with evidence).
- Material misrepresentation by counterparty.
- Documented force majeure.
- Counterparty's prior pledge violation.

Anything else is "without cause" by default.

---

## Reviews

### Mechanics
- Available at the close of a hiring pipeline (rejected, withdrawn, or hired).
- 1–5 stars + free-text comment.
- Subject is notified and given 24 hours of Right of Reply before public publish.
- Reviews can be challenged via Dispute.

### Weighting
- A reviewer's own reputation weights their reviews. A Platinum-tier candidate's review of a company counts more than a brand-new account's.
- Brigading detection: if a company suddenly gets >3 negative reviews in a week from low-reputation accounts, the system holds them for review.

### Deletion
- Only via Dispute resolution.
- Right of Reply persists even after deletion.

---

## Strikes & Suspensions

### Strikes
- Issued by Admin, never automatic.
- Reasons enumerated: AUP violation, harassment, fraud, reputation gaming, repeated pledge violations.
- 3 strikes within 12 months → 30-day suspension.
- 5 strikes ever → permanent ban (with appeal).

### Appeals
- One appeal per strike.
- Decided by a different Admin from the issuer.

---

## Disputes

### Triggerable on
- Pledge violation.
- Review.
- Submission evaluation.
- Escrow release / refund.
- Strike issuance.

### Process
1. Filed in-app with structured form.
2. Auto-acknowledgement within 5 minutes.
3. Triage by support within 24 h.
4. Mediation by Admin within 5 business days.
5. Decision: upheld / overturned / partial.
6. Reputation events updated.
7. Right of single appeal within 7 days.

### Evidence the platform retains
- Pledge metadata.
- Sandbox event logs.
- Pipeline timeline.
- Message history (where applicable).
- Audit log of all actions.

---

## Fraud & Integrity Layer

JobCeption runs a dedicated Fraud & Integrity Service. It is critical infrastructure on a competitive platform — the assumption is that bad actors will try, not might.

### Identity-Tied Defenses
- **KYC for high-value bounties.** Identity verification (Persona / Onfido / Hyperverge — see `03-integration-spec.md`) is mandatory for any freelance bounty above a configurable reward threshold and for any candidate issuing a Pledge.
- **Device fingerprinting.** Multiple accounts on the same device fingerprint flagged for review. Not auto-banned (legitimate shared-device use cases exist), but a strong signal.
- **Account age and reputation gating.** New accounts cannot participate in high-value bounties until a cooldown + minimum AP threshold is met.

### Behavioral Biometrics (Anti-Cheat)
A keystroke dynamics logger runs client-side during sandbox sessions. It captures:
- **Inter-keystroke timing distribution** (sliding window).
- **Paste event size, frequency, and timing relative to typing bursts.**
- **Sudden velocity changes** (e.g., 100 chars in 200ms after 30s of slow typing).
- **Typing rhythm fingerprint** — used to detect when "the same person" is supposedly working two accounts.

These signals are stored as `anti_cheat_flags` on the submission. They are NEVER auto-disqualifying — they raise the submission's review priority in the admin queue.

### Code Plagiarism Detection
Two-tier:
1. **MOSS (Stanford Measure Of Software Similarity)** — the academic-grade algorithm for code similarity, specifically designed to be resilient to renaming, reordering, and superficial obfuscation. Either invoked via the public MOSS service or reimplemented in-house (~few hundred lines of Python).
2. **MinHash / LSH (Locality Sensitive Hashing)** — a custom near-duplicate detector that runs in real-time at submission. Faster than MOSS but less precise; serves as the first-pass filter that triggers MOSS for borderline cases.

Both run against the corpus of all submissions to the same bounty. Above-threshold similarity creates an admin queue item; submission is NOT auto-disqualified.

### Text Plagiarism Detection
For writing-bounty submissions: sentence-transformer embeddings + cosine similarity against past submissions. Same flag-and-queue pattern.

### AI-Generated Content Detection
- **Code:** `roberta-base-openai-detector` from Hugging Face, self-hosted on CPU. Output is a probability score 0–1, stored as `ai_content_score` in `anti_cheat_flags`. NOT a verdict.
- **Writing:** same model, weighted higher in the report because writing bounties typically expect human work.
- **Limitations:** these detectors are imperfect and degrade over time as models improve. The score is a signal in the report, not a basis for action.

### Review Brigading Detection
- **Velocity check:** if a company suddenly receives more than 3 negative reviews in a week from low-reputation accounts, the reviews are held for admin review before publishing.
- **Network graph check:** reviewers who consistently rate together (positively or negatively) flagged for collusion review.
- **Reviewer reputation weighting** (already in scoring): a brigade of low-reputation accounts has minimal effect on aggregate score even if not caught.

### Cross-Cutting Defenses

| Vector | Defense |
|---|---|
| Sock-puppet endorsements | Per-peer cap (5/quarter); identity verification gating |
| Multiple accounts gaming bounties | Device fingerprinting + keystroke biometrics + KYC for high-value |
| Code submission cheating | MOSS + MinHash/LSH + RoBERTa AI-content detector + behavioral biometrics |
| Text submission cheating | Sentence-transformer plagiarism + RoBERTa detector + LanguageTool anomaly |
| Coordinated review brigading | Velocity detection + network graph + manual hold |
| Reputation farming via easy bounties | Diminishing returns per template; difficulty scaling |
| Recruiter inflating Pledge Honor by hiring fakes | Hire-completion verification (joining date proof, KYC) |
| Badge farming via paid networks | Badges scaled by reviewer reputation; AML on payments |
| Leaderboard probing on data science bounties | Public/private split (Kaggle pattern) — see `03-evaluation-spec.md` |

---

## Materialization
- The `reputation_event` table is the source of truth.
- A nightly job recomputes `xp_snapshot` and `reputation_score`.
- Hot-path reads use the snapshot, never the ledger.
- Recomputation is idempotent — formulas can be re-tuned and historical scores re-derived (ADR-0006 governs how this is communicated to users).

---

## Open Questions
- How quickly should pledge violations decay from the reputation score?
- Should we publish a public "leaderboard" or keep XP a personal stat?
- Is the dilution ratio user-visible or hidden behind the display XP number?
- Can a user purchase a "reputation reset" after N years?
- How do we handle reputation portability — can a user export their score elsewhere?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
- 2026-04-10 — v0.2 — Expanded anti-gaming into a full Fraud & Integrity Layer section. Added behavioral biometrics, MOSS, MinHash/LSH, RoBERTa detector, brigading detection.
