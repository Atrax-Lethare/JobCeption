# Domain Model

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Tech Lead
- **Last Updated:** 2026-04-09
- **Depends On:** 00-glossary.md, 01-functional-requirements.md
- **Referenced By:** 02-database-spec.md, 03-api-spec.md, 03-ai-intelligence-spec.md, 09-trust-and-reputation-spec.md, 09-analytics-spec.md

## Purpose
Define the entities, relationships, invariants, and lifecycles that constitute the JobCeption domain. This is the shape of truth for the entire system; database, API, and analytics specs are all derived from it.

## Modeling Conventions
- **Identifier convention:** every entity has a server-generated UUID v7 primary key (`id`).
- **Soft deletes:** all entities use `deleted_at` (nullable timestamp). Hard delete only on legal request.
- **Audit fields:** `created_at`, `updated_at`, `created_by`, `updated_by` on every mutable entity.
- **Money fields:** stored as `(amount_minor_units BIGINT, currency CHAR(3))`. No floats.
- **Timestamps:** UTC, millisecond precision.
- **Enum fields:** stored as VARCHAR with CHECK constraint, enumerated here.
- **Invariants:** statements that must always be true. Enforced at the database level where possible, in service code otherwise.

---

## Entity Catalogue

### 1. User
The base identity entity. A single human may own one Candidate identity AND one Recruiter identity, but they are separate `user` rows linked by `linked_user_id`.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| email | varchar(320) | unique, lowercase |
| password_hash | varchar | nullable (passkey users) |
| user_kind | enum | `candidate`, `recruiter`, `admin` |
| status | enum | `active`, `suspended`, `deactivated` |
| verified_identity | bool | from KYC flow |
| linked_user_id | uuid | nullable, points to other identity |
| created_at, updated_at | timestamp | |

**Invariants:**
- `email` is globally unique.
- A user with `user_kind = recruiter` MUST belong to at least one `company` (via `company_member`).
- A `suspended` user cannot publish bounties or submit pledges.

**Lifecycle:** `pending_verification → active → (suspended | deactivated)`.

---

### 2. CandidateProfile
1:1 with a `candidate` user.

| Field | Type | Notes |
|-------|------|-------|
| user_id | uuid | PK, FK → user |
| display_name | varchar(80) | |
| username | varchar(40) | unique, immutable after first set |
| tagline | varchar(160) | |
| avatar_url | varchar | |
| avatar_frame_id | uuid | nullable |
| availability | enum | `open_to_bounties`, `open_to_offers`, `not_looking` |
| incognito_company_ids | uuid[] | companies blocked from seeing activity |
| location_country, location_city | varchar | |

---

### 3. Company
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| legal_name | varchar | |
| display_name | varchar | |
| domain | varchar | verified email domain |
| verified | bool | |
| rating | numeric(3,2) | 0.00–5.00, materialized from reviews |
| pledge_honor_rate | numeric(5,4) | 0.0000–1.0000 |
| created_at | timestamp | |

**Invariants:**
- `rating` and `pledge_honor_rate` are materialized; never written by hand.
- A company without `verified = true` cannot publish job bounties.

---

### 4. CompanyMember
Join table between `user` and `company`.

| Field | Type | Notes |
|-------|------|-------|
| user_id | uuid | FK → user |
| company_id | uuid | FK → company |
| role | enum | `owner`, `hiring_manager`, `recruiter` |
| status | enum | `active`, `removed` |

**Invariants:** at least one `owner` per company at all times.

---

### 5. Bounty
The atomic unit of work.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| company_id | uuid | FK |
| created_by_user_id | uuid | FK |
| title | varchar(120) | |
| bounty_type | enum | `job`, `freelance`, `xp`, `competition` |
| status | enum | `draft`, `published`, `closed`, `archived`, `disputed` |
| description_md | text | |
| requirements_md | text | |
| salary_min, salary_max | bigint | minor units, nullable for non-job |
| salary_currency | char(3) | |
| reward_amount | bigint | nullable, freelance/competition |
| reward_currency | char(3) | |
| location_kind | enum | `remote`, `onsite`, `hybrid` |
| location_country, location_city | varchar | nullable |
| start_date | date | nullable |
| template_id | uuid | FK → bounty_template |
| applications_open_at, applications_close_at | timestamp | |
| published_at | timestamp | nullable |
| escrow_transaction_id | uuid | nullable |

**Invariants:**
- Once `status = published`, `bounty_type`, `salary_min`, `salary_max`, `reward_amount`, `description_md`, and `requirements_md` become immutable. Edits are recorded as `bounty_amendment` rows and cost reputation.
- `bounty_type = job` REQUIRES `salary_min` and `salary_max` set.
- `bounty_type = freelance` REQUIRES a successfully escrowed `escrow_transaction_id`.
- `applications_close_at > applications_open_at`.

**Lifecycle:** `draft → published → closed → (archived | disputed)`.

---

### 6. BountyConstraint
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| bounty_id | uuid | FK |
| kind | enum | `min_xp`, `language`, `framework`, `time_complexity`, `space_complexity`, `geography`, `min_company_rating`, `custom` |
| value | jsonb | shape depends on kind |
| visibility | enum | `visible`, `hidden` |

---

### 7. BountyRound
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| bounty_id | uuid | FK |
| round_index | int | 1-based |
| name | varchar | |
| starts_at, ends_at | timestamp | |
| rubric_md | text | evaluation criteria |
| max_score | int | |

**Invariants:** rounds are gapless from `round_index = 1`.

---

### 8. BountyTemplate
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| kind | enum | `platform`, `creative` |
| owner_company_id | uuid | nullable for platform templates |
| name | varchar | |
| domain_tags | varchar[] | |
| structure_jsonb | jsonb | round definitions, default constraints |
| review_status | enum | `pending`, `approved`, `rejected` (creative only) |

---

### 9. BountyParticipation
A candidate's enrollment in a bounty.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| bounty_id, candidate_user_id | uuid | FK |
| layer_of_interest | enum | `for_job`, `for_xp`, `for_reward` |
| joined_at | timestamp | |
| status | enum | `joined`, `submitted`, `withdrawn`, `disqualified`, `completed` |
| withdrawn_reason | varchar | nullable |

**Invariants:**
- `(bounty_id, candidate_user_id)` is unique.
- Setting `status = withdrawn` after `layer_of_interest = for_job` triggers a `reputation_event`.

---

### 10. BountySubmission
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| participation_id, round_id | uuid | FK |
| sandbox_session_id | uuid | FK |
| submitted_at | timestamp | |
| artifact_storage_url | varchar | immutable object storage URL |
| sha256 | char(64) | content hash |
| anti_cheat_flags | jsonb | tab switches, paste detections, AI score |
| status | enum | `submitted`, `evaluated`, `disputed` |

**Invariants:** once `submitted`, the artifact is immutable.

---

### 11. SubmissionEvaluation
Two evaluations per submission: one platform, one recruiter.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| submission_id | uuid | FK |
| evaluator_kind | enum | `platform`, `recruiter` |
| evaluator_user_id | uuid | nullable for platform |
| score | int | |
| max_score | int | |
| breakdown_jsonb | jsonb | per-criterion |
| feedback_md | text | |
| evaluated_at | timestamp | |

**Invariants:** exactly one `platform` evaluation per submission; recruiter evaluations optional.

---

### 12. HiringPipeline
The post-bounty interview process.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| bounty_id, candidate_user_id, company_id | uuid | FK |
| status | enum | `active`, `offered`, `pledged_by_company`, `pledged_by_candidate`, `hired`, `rejected`, `abandoned`, `disputed` |
| started_at | timestamp | |

---

### 13. PipelineStage
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| pipeline_id | uuid | FK |
| stage_index | int | |
| name | varchar | |
| scheduled_for | timestamp | |
| completed_at | timestamp | nullable |
| sla_deadline | timestamp | |
| delay_flagged | bool | |

---

### 14. DocumentRequest
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| pipeline_id | uuid | FK |
| document_kind | varchar | e.g. `degree_certificate`, `id_proof` |
| due_at | timestamp | |
| uploaded_at | timestamp | nullable |
| storage_url | varchar | nullable |
| status | enum | `pending`, `uploaded`, `verified`, `rejected`, `overdue` |

---

### 15. Pledge
The legally significant binding action.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| pipeline_id | uuid | FK |
| pledge_kind | enum | `recruiter_offer`, `candidate_acceptance` |
| issued_by_user_id | uuid | FK |
| issued_at | timestamp | immutable |
| terms_jsonb | jsonb | snapshot of salary, start date, role |
| ip_address, user_agent | varchar | for evidentiary record |
| outcome | enum | `pending`, `honored`, `withdrawn_with_cause`, `withdrawn_without_cause` |
| outcome_decided_at | timestamp | nullable |

**Invariants:**
- A pledge is immutable except for `outcome` and `outcome_decided_at`.
- A `recruiter_offer` must precede a `candidate_acceptance` on the same pipeline.
- `withdrawn_without_cause` triggers a `reputation_event` with `severity = high`.

---

### 16. Review
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| pipeline_id | uuid | FK |
| author_user_id | uuid | FK |
| subject_kind | enum | `candidate`, `company` |
| subject_id | uuid | FK |
| rating | int | 1–5 |
| comment_md | text | |
| reply_md | text | nullable, single Right of Reply |
| status | enum | `published`, `disputed`, `removed` |
| created_at | timestamp | |

**Invariants:** one review per (pipeline, author, subject).

---

### 17. ReputationEvent
The append-only ledger that drives reputation scores.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| subject_user_id, subject_company_id | uuid | one of, FK |
| event_kind | enum | see catalogue in `09-trust-and-reputation-spec.md` |
| delta_aura, delta_creative_aura | int | signed |
| severity | enum | `low`, `medium`, `high` |
| source_id | uuid | the entity that caused this (pledge, review, submission) |
| created_at | timestamp | |

**Invariants:** append-only; no updates, no deletes.

---

### 18. XPSnapshot
Materialized view of total AP and CAP per user, refreshed on each `reputation_event`.

---

### 19. Post
Connections content.

| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| author_user_id, author_company_id | uuid | one of |
| body_md | text | |
| attachments_jsonb | jsonb | |
| visibility_score | numeric | computed |
| ai_slop_score | numeric | 0.0–1.0 |
| status | enum | `published`, `quarantined`, `removed` |
| created_at | timestamp | |

---

### 20. BadgeAward
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| post_id | uuid | FK |
| awarded_by_user_id | uuid | FK |
| badge_kind | varchar | |
| paid_amount, currency | money | |

---

### 21. EscrowTransaction
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| bounty_id | uuid | FK |
| amount, currency | money | |
| state | enum | `held`, `released`, `refunded`, `disputed` |
| provider | varchar | e.g. `stripe_connect` |
| provider_reference | varchar | external ID |
| created_at, settled_at | timestamp | |

---

### 22. Dispute
| Field | Type | Notes |
|-------|------|-------|
| id | uuid | PK |
| filed_by_user_id | uuid | FK |
| subject_kind | enum | `pledge`, `review`, `evaluation`, `escrow` |
| subject_id | uuid | |
| reason_md | text | |
| status | enum | `open`, `under_review`, `resolved_upheld`, `resolved_overturned`, `resolved_partial` |
| resolution_md | text | |
| filed_at, resolved_at | timestamp | |

---

### 23. AuditLog
Append-only log of every privileged action.

---

## Relationship Diagram (text)
```
User ─┬─ CandidateProfile
      └─ CompanyMember ── Company ── Bounty ── BountyRound
                                       │       └ BountyConstraint
                                       └─ BountyParticipation ── BountySubmission ── SubmissionEvaluation
                                                                         └ HiringPipeline ── PipelineStage
                                                                                              └ DocumentRequest
                                                                                              └ Pledge
                                                                                              └ Review
                                       └─ EscrowTransaction
ReputationEvent (subject = User | Company)  ←  triggered by Pledge, Review, Submission, etc.
```

## Cross-Cutting Invariants
1. **Pledge integrity:** a `Pledge` row, once written, is mutable only on `outcome` fields. Enforced by DB row trigger.
2. **Bounty immutability after publish:** a `bounty.status` transition to `published` freezes specified fields. Enforced by row trigger.
3. **Reputation append-only:** `reputation_event` rejects updates and deletes at the DB level.
4. **Company-recruiter membership:** a recruiter user cannot publish a bounty without an active `company_member` row.
5. **Escrow before freelance publish:** `bounty.status = published AND bounty_type = freelance` is rejected unless `escrow_transaction_id` is set and `state = held`.

## Open Questions
- Do we need a separate `Offer` entity between `HiringPipeline` and `Pledge`, or are pledges enough?
- How do we model team bounties — as a `BountyTeam` join table or as parent-child bounties?
- Should `ReputationEvent` carry a `reversible` flag for events that can be unwound by appeal?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
