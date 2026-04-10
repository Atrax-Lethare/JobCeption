# Recommendation & Search Spec

- **Version:** 0.2
- **Status:** Draft
- **Owner:** AI / Backend
- **Last Updated:** 2026-04-10
- **Depends On:** 02-domain-model.md, 03-ai-intelligence-spec.md
- **Referenced By:** 09-analytics-spec.md, 06-frontend-spec.md

## Purpose
Define how JobCeption surfaces the right bounty to the right candidate, the right candidate to the right recruiter, and the right job to the right candidate. Three related systems live here:

1. **Bounty Recommendation** — for candidates browsing the marketplace.
2. **Recruiter Search** — for recruiters finding candidates via the Dynamic Profile.
3. **Candidate ↔ JD Matching** — relevance score between a posted JD and a candidate's portfolio.

All three rely on shared embedding infrastructure and the same vector database.

## Shared Infrastructure

### Vector Database: Weaviate (self-hosted)
- Deployed on EKS as a stateful service.
- Stores three vector collections: `bounty_vectors`, `candidate_vectors`, `post_vectors`.
- Per-collection HNSW indexing.
- Backed up to S3 nightly.
- Fallback: Pinecone free tier as a read-only DR mirror.

### Embedding Model
- `sentence-transformers/all-MiniLM-L6-v2`, self-hosted on CPU via FastAPI behind the AI Gateway.
- 384-dimensional output.
- ~50ms inference on CPU per item; bulk reindex via batching.

### Reindex Strategy
- **On write:** every new or updated bounty / profile / post enqueues a reindex job.
- **Nightly batch:** full reindex of any item touched in the last 24h to catch missed events.
- **Embedding model upgrade:** triggers a full reindex; new vectors written to a new collection, traffic cut over with a feature flag, old collection retained for 7 days.

---

## System 1: Bounty Recommendation

### Goal
For each candidate visiting the marketplace, return a personalized ranked list of bounties they are likely to find relevant and likely to complete successfully.

### Algorithm: Hybrid (Content + Collaborative)
A hybrid model is used because pure collaborative filtering fails on cold-start (new candidates, new bounties), and pure content-based filtering ignores behavioral signal.

### Component A: Content-Based (cold-start friendly)
For each candidate, build a **candidate vector** from:
- Self-declared skills (text).
- Domain interests (text).
- Salary expectation (numeric, normalized).
- Location (categorical).
- Top 5 completed bounty titles (text).

For each bounty, build a **bounty vector** from:
- Title + description + requirements (text → embedding).
- Constraints (categorical).
- Salary range (numeric, normalized).

Similarity: cosine between candidate vector and bounty vector. Top-K returned.

### Component B: Collaborative Filter
Trained periodically (nightly batch) using **LightFM**. Inputs:
- Implicit signals: bounty_viewed, bounty_joined, bounty_completed.
- Explicit signals: bounty_saved, bounty_shared.
- User features: same as the candidate vector above.
- Item features: same as the bounty vector above.

LightFM is chosen because it natively supports hybrid (item + user features alongside the interaction matrix), which gives us a graceful cold-start fallback inside a single model.

### Component C: Filters and Business Rules
Hard filters applied AFTER scoring:
- Bounty status = published, applications open.
- Candidate meets the bounty's declared minimum AP.
- Candidate's location matches if bounty is location-restricted.
- Candidate's incognito-blocked companies are excluded.

### Combination
`final_score = α × content_score + β × collab_score + γ × recency_boost`

Default weights: α=0.4, β=0.4, γ=0.2. Tunable via config and re-tuned via online A/B.

### Cold-Start Behavior
- **New candidate, no history:** content-based only, weighted toward declared interests.
- **New bounty, no joins:** content-based only, weighted toward template similarity to past popular bounties.

### Eval
- Offline: nDCG@10 on a held-out interaction set. Target ≥ 0.65 at MVP.
- Online: A/B test recommendation variants by completion rate of recommended bounties.

### Fallback
If LightFM training pipeline fails or the vector DB is down:
1. Content-only ranking.
2. Recency sort (newest first).
3. Static popular list.

---

## System 2: Recruiter Search

### Goal
A recruiter types a natural-language query ("senior backend engineer, Go, payments experience, Bengaluru") and gets a ranked list of relevant candidates.

### Architecture
```
Recruiter Query (text)
    │
    ▼
AI Gateway → sentence-transformers (embed query)
    │
    ▼
Weaviate (kNN search on candidate_vectors, top 100)
    │
    ▼
Cross-encoder reranker (cross-encoder/ms-marco-MiniLM-L-6-v2)
    │  scores each (query, candidate_summary) pair
    ▼
Hard filters (skill tags, AP threshold, availability, geography)
    │
    ▼
Final ranked list (top 20 to UI)
```

### Candidate Vector
Built from the candidate's portfolio summary (see `09-social-spec.md` and the Dynamic Profile spec). Inputs:
- Auto-generated CV text.
- Top completed bounty titles.
- Top platform-evaluated strengths.
- Self-declared skills.
- Aura Points and Reputation Score (NOT embedded — used as filters/boosts).

### Reranking
The cross-encoder model `cross-encoder/ms-marco-MiniLM-L-6-v2` is run on the top 100 kNN results to rescore (query, candidate) pairs with much higher precision than embedding cosine alone. This is the standard "retrieve-then-rerank" pattern and dramatically improves relevance.

### Filters (deterministic)
After AI scoring, hard filters apply:
- Skill tag intersection.
- Minimum AP threshold.
- Availability status (`open_to_offers` or `open_to_bounties`).
- Geography.
- Excludes candidates in incognito mode against the searching company.

### Performance
- Target query latency: < 800ms p95.
- Achieved by limiting the kNN-then-rerank pipeline to top 100 candidates.

### Privacy
- A search query that returns zero results does not reveal whether a candidate exists but is hidden by incognito — the response is identical to "not found".
- Searches are logged for audit but never visible to candidates.

### Fallback
1. If reranker is down: return kNN results without reranking.
2. If Weaviate is down: fall back to Postgres full-text search on candidate summaries.
3. If both are down: filter-only search.

---

## System 3: Candidate ↔ JD Matching Score

### Goal
For a given JD and a given candidate, compute a single relevance score (0–100). Used in two places:
1. The recruiter's submissions screen, to highlight high-match candidates among the bounty applicants.
2. The candidate's bounty detail page, to show "your match score" before they commit.

### Algorithm
Run the cross-encoder `cross-encoder/ms-marco-MiniLM-L-6-v2` on the (jd_text, candidate_summary) pair. Output: a relevance probability 0–1.

`match_score = round(probability × 100)`

### What it includes
- Skill overlap (implicit in the embedding).
- Experience seniority match.
- Domain relevance.

### What it does NOT include
- Demographic features (illegal and forbidden).
- Salary fit (handled by hard filter, not embedded).
- Location fit (handled by hard filter).
- Reputation Score (visible separately, not blended in).

### Display
- Shown as a single number with a tooltip explaining "based on JD ↔ portfolio similarity. Does NOT factor in salary, location, or reputation — those are shown separately."
- Color-coded: green ≥ 80, yellow 60–79, gray < 60.

### Eval
Offline: labeled set of (jd, candidate, recruiter_judgment) tuples. Target: nDCG@10 ≥ 0.80.

---

## Notification Trigger
When a new bounty is published:
1. Bounty Service emits `bounty_published` event.
2. Recommendation Service consumes asynchronously.
3. For each active candidate whose vector predicts high relevance (above threshold), enqueue a notification.
4. Notification Service rate-limits to at most 5 bounty alerts per candidate per day.

The trigger uses the same recommendation pipeline as the marketplace browse path — there's only one ranking model, used in two places.

---

## Real-time Layer
- **WebSocket connections** are used for live competition events (leaderboards, new submission notifications during a bounty round). Implementation: Socket.io on the Notification Service.
- **Async alerts** (new matching bounty published, new matching candidate available) flow through the Redis Streams queue → Notification Service → email/in-app/push.

## Open Questions
- LightFM retraining cadence — nightly is conservative; can we do incremental updates?
- Should the candidate vector include negative signal (bounty types they declined)?
- Cross-encoder reranking on top 100 is the bottleneck — should we cap at top 50 for sub-500ms p95?
- Do we expose the JD ↔ candidate match score to the candidate at all, or only to the recruiter?

## Change Log
- 2026-04-10 — v0.1 — Initial draft.
