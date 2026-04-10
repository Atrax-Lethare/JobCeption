# Evaluation Spec

- **Version:** 0.2
- **Status:** Draft
- **Owner:** Tech Lead / AI
- **Last Updated:** 2026-04-10
- **Depends On:** 02-domain-model.md, 03-ai-intelligence-spec.md, 03-sandbox-execution-spec.md
- **Referenced By:** 09-trust-and-reputation-spec.md, 09-analytics-spec.md

## Purpose
Define how submissions are evaluated, scored, and ranked, per bounty type. Evaluation is the bridge between raw work output and the reputation economy — it must be fair, deterministic where possible, transparent in its reasoning, and resistant to gaming. This is a flagship subsystem.

## Principles
1. **Per-type pipelines.** There is no single "evaluator". Each bounty type has its own pipeline tuned to that domain.
2. **Two evaluations per submission.** Every submission gets a **Platform Evaluation** (deterministic, drives Aura Points) AND a **Recruiter Evaluation** (subjective, drives hiring decisions only). These NEVER influence each other.
3. **Deterministic core, AI assistance.** Every pipeline has a deterministic component that produces a defensible score. AI components add nuance but can never be the sole signal.
4. **Transparent reasoning.** Every score is decomposable into criteria; every criterion is explainable.
5. **Hidden test isolation.** Test cases, ground truth, and rubrics live server-side and are never sent to the candidate.
6. **Ranking is separate from scoring.** Scores are absolute (0–100); ranks are relative (within the bounty cohort).

## Architecture

```
BountySubmission (status: submitted)
       │
       ▼
Evaluation Service (router by bounty_type)
       │
       ├─► Code Pipeline ──► SubmissionEvaluation (platform)
       ├─► Design Pipeline ──► SubmissionEvaluation (platform)
       ├─► Writing Pipeline ──► SubmissionEvaluation (platform)
       └─► Data Science Pipeline ──► SubmissionEvaluation (platform)
                                              │
                                              ▼
                                      Ranking Service (per round)
                                              │
                                              ▼
                                      Performance Report
                                              │
                                              ▼
                                      Reputation Event
```

A separate Recruiter Evaluation is written by the recruiter at their own pace; it does not block any of the above.

---

## Pipeline A: Code / Technical Bounties

### Inputs
- Source code (one or more files).
- Language declared by the bounty template.
- Hidden test suite from the bounty.
- Optional: starter code, reference solution.

### Steps

**1. Compile / parse**  
Submitted via Judge0 with a no-op test to confirm it compiles. Compile failures get score 0 + a clear error in the report.

**2. Functional tests (deterministic)**  
Judge0 runs all hidden test cases sequentially. For each: pass/fail, runtime, peak memory.  
**Score:** `passed_cases / total_cases × 60` (max 60 points).

**3. Performance metrics (deterministic)**  
For passed cases, compute median runtime and median memory.  
**Score:** linear scale against the bounty's declared performance bounds: 20 points if runtime ≤ best 25%, 0 if > worst 25%, linear in between.

**4. Code quality (deterministic + tooled)**  
Source piped through SonarQube CE. Returns: bug count, code smell count, security issue count, maintainability rating.  
**Score:** 10 points minus penalties (bugs −2 each, security issues −3 each, smells −0.5 each, capped at 0).

**5. AI-content check (signal, not score)**  
Source run through `roberta-base-openai-detector`. Output stored in `anti_cheat_flags.ai_content_score`. Does NOT directly affect the score; surfaces in the report and admin tools.

**6. Plagiarism check (signal + admin escalation)**  
Source compared against all other submissions to the same bounty using MOSS + MinHash/LSH. If similarity > threshold, an admin queue item is created. Submission is NOT auto-disqualified.

**7. Final score**  
`code_score = functional + performance + code_quality` (max 90; remaining 10 points reserved for the recruiter evaluation if they choose to add a manual score, otherwise 0).

### Example Output
```json
{
  "score": 78,
  "max_score": 100,
  "breakdown": {
    "functional": { "passed": 9, "total": 10, "score": 54 },
    "performance": { "median_runtime_ms": 120, "score": 18 },
    "code_quality": { "bugs": 0, "smells": 4, "security": 0, "score": 6 },
    "anti_cheat": { "ai_content_score": 0.12, "plagiarism_max": 0.34 }
  },
  "feedback_md": "Strong functional correctness — 9/10 cases passed. Edge case for empty input fails. Performance is in the top quartile. Minor code smells around variable naming."
}
```

---

## Pipeline B: Design / Creative Bounties

Design bounties cannot be evaluated by deterministic code. JobCeption uses a **structured peer review system with AI assistance**.

### Inputs
- Submitted artifact (image, Figma file URL, video).
- Recruiter's brief and (optional) reference aesthetic samples or brand guidelines.

### Steps

**1. Format validation (deterministic)**  
File type, dimensions, file size match bounty constraints. Score 0 if not.

**2. AI-assisted aesthetic match (signal)**  
Submission and reference samples embedded with `openai/clip-vit-base-patch32`. Cosine similarity computed.  
**Score:** 0–30 points scaled from similarity, with a floor of 5 to avoid penalizing legitimately divergent design.

**3. Brief adherence (signal)**  
Brief and submission caption / accompanying notes embedded with sentence-transformers. Cosine similarity → 0–20 points.

**4. Peer review (the dominant signal)**  
The bounty enters a structured peer review phase:
- 5–10 reviewers are selected, weighted toward higher-AP / higher-Reputation accounts in the same domain.
- Each reviewer rates the submission across declared criteria (e.g., visual hierarchy, originality, brief adherence).
- Reviewer scores are aggregated using **weighted average**, where weight = reviewer's Reputation Score / 100.
- Outliers (>2 stddev from mean) flagged but not removed.
- **Score:** 0–50 points from the weighted average, normalized.

**5. Final score**  
`design_score = aesthetic + brief + peer_review` (max 100).

### Anti-collusion
- Reviewers are randomized from the pool, never chosen by the candidate or recruiter.
- A reviewer cannot review more than N submissions per bounty.
- Reviewers cannot see each other's scores until after they submit.
- Same-company collusion check: reviewers from the recruiter's company are excluded.

---

## Pipeline C: Writing / Documentation Bounties

### Inputs
- Submission text.
- Bounty brief / rubric.
- Optional: reference exemplars.

### Steps

**1. Length and format check (deterministic)**  
Word count within bounds, required sections present (if specified). Score 0 if missing required sections.

**2. Brief adherence (semantic similarity)**  
Submission and brief embedded with sentence-transformers. Cosine similarity → 0–25 points.

**3. Quality dimensions (zero-shot classification)**  
Submission run through `facebook/bart-large-mnli` via HF Inference API with declared quality dimensions (clarity, completeness, accuracy, structure). Each returns a 0–1 score.  
**Score:** average × 35 (max 35 points).

**4. Grammar and style (LanguageTool)**  
Self-hosted LanguageTool runs on the submission. Errors counted, weighted by category (grammar > style > typo).  
**Score:** 20 points minus penalties (capped at 0).

**5. AI-content check (signal)**  
RoBERTa detector. Stored in flags. For writing bounties this signal is **higher-weighted** in the report — recruiters typically want human writing.

**6. Plagiarism check (sentence-transformers cosine against past submissions)**

**7. Optional human review for borderline cases**  
If `(bart_score < 0.5 OR ai_content_score > 0.7)`, queue for human review before finalizing.

**8. Final score**  
`writing_score = brief + quality + grammar` (max 80; reserve 20 for recruiter evaluation).

---

## Pipeline D: Data Science / ML Bounties

Kaggle-style leaderboard evaluation.

### Inputs
- Submission file (predictions in a declared format) OR a serialized model.
- Hidden ground truth dataset.
- Declared metric (F1, RMSE, AUC, accuracy, etc.).

### Steps

**1. Format validation (deterministic)**  
File parses, has the right number of rows, columns match expected schema.

**2. Metric computation (deterministic)**  
Predictions compared against hidden ground truth. Metric computed.  
**Score:** scaled to 0–80 against the bounty's declared baseline (baseline = 0, top theoretical = 80) OR percentile among all submissions, recruiter's choice.

**3. Submission integrity check (deterministic)**  
Detect overfitting to a public set: predictions evaluated against TWO hidden sets — primary and validation. Large gap between the two flags potential leaderboard probing.

**4. (Optional) Code review of training pipeline**  
If the bounty requires a reproducible pipeline, the submitted code goes through Pipeline A (Code) for an additional quality score (0–20 points).

**5. Plagiarism check (model output similarity)**  
Submitted predictions embedded; cosine similarity against all other submissions. Identical predictions → admin queue.

**6. Final score**  
`ds_score = metric + (optional code_quality)` (max 100).

### Leaderboard
- Live during the bounty (encourages competition), but the leaderboard reflects only the **public split** of the hidden set; the **final** score uses the **private split** and is computed once after the bounty closes. Standard Kaggle pattern, prevents leaderboard overfitting.

---

## Ranking (post-evaluation)

After all submissions to a bounty round are scored, the Ranking Service computes a final ranked list.

### For single-criterion bounties
Sort by score, descending. Tiebreak by submission time (earlier wins).

### For multi-criteria bounties
Use **TOPSIS** (Technique for Order of Preference by Similarity to Ideal Solution) — a multi-criteria decision method that produces a single rank from multiple weighted criteria. Implementation: small in-house Python module.

### Skill rating updates
For competitive bounties, candidates' skill ratings are updated using **TrueSkill** (Microsoft's Bayesian rating system). Implementation: `trueskill` Python library. TrueSkill ratings are stored on the candidate profile and contribute to the difficulty calibration of future bounties.

### Output
The ranked list, with each candidate's:
- Final score (0–100).
- Rank (1 to N).
- Percentile.
- TrueSkill rating delta.
- Per-criterion breakdown.

---

## Performance Report

Generated for every candidate after every round, regardless of outcome. Contains:
1. **Score ring** (0–100).
2. **Constraint satisfaction checklist.**
3. **Per-criterion breakdown** with explanations.
4. **Strengths and weaknesses** — generated from the breakdown by template (no LLM needed for MVP) or by `facebook/bart-large-cnn` if a richer narrative is desired.
5. **Ranking percentile** (anonymized — never reveals other candidates).
6. **Anti-cheat flags summary** (only if non-trivial — visible to candidate so they can dispute).
7. **Next steps** (template-driven recommendations).

The Performance Report is THE anti-ghosting commitment of the platform. It is generated even if the candidate finished last.

---

## SLA
- Code, writing, data science bounties: report within **15 minutes** of submission close.
- Design bounties (require peer review): report within **72 hours** of peer review window close.
- Custom Docker bounties: report within **30 minutes** of submission close.

Missing the SLA generates a Delay Flag on the platform itself (not on the recruiter or candidate).

## Open Questions
- For design bounties, what's the minimum number of reviewers before peer-review aggregation is statistically valid?
- Should we expose the hidden test results to candidates after the bounty closes (educational) or never (preserves test integrity)?
- TrueSkill vs. Glicko-2 for skill rating — TrueSkill handles team contexts better but Glicko-2 has simpler tuning.

## Change Log
- 2026-04-10 — v0.1 — Initial draft.
