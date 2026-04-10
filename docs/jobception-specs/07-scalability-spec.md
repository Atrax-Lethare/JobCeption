# Scalability Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Tech Lead
- **Last Updated:** 2026-04-09
- **Depends On:** 01-non-functional-requirements.md
- **Referenced By:** —

## Purpose
Define how the system scales horizontally, where caching lives, queue patterns, and where we expect bottlenecks.

## Scaling Patterns
- **Stateless services:** horizontal autoscaling on CPU + custom metrics (queue depth, p95 latency).
- **Stateful sandbox workers:** pre-warmed pool + HPA on queue depth.
- **DB:** vertical scale + read replicas; sharding deferred.
- **Search:** Meilisearch cluster with read replicas.

## Caching
- **Edge:** CloudFront for marketing pages.
- **API:** Redis for bounty marketplace listings (60 s TTL), session data, rate limit counters.
- **Application:** in-process LRU for hot configs.

## Queues
- Redis Streams or NATS.
- Topics: `notifications`, `analytics`, `reputation_events`, `search_index`, `email`, `webhook_outbound`.
- DLQ per topic; alarms on DLQ depth.

## Hot Paths
| Path | Concern | Mitigation |
|---|---|---|
| Marketplace browse | Read amplification | Search + cache |
| Pledge write | Latency, durability | Dedicated service, sync writes |
| Sandbox provisioning | Cold-start latency | Warm pool |
| Reputation materialization | Write amplification | Async pipeline |

## Bottlenecks We Expect
- Sandbox during bounty deadlines.
- DB writes during pipeline activity bursts.
- Notification fanout for popular Connections posts.

## Open Questions
- When to introduce Kafka.
- Whether to front the DB with a connection pooler (PgBouncer / RDS Proxy) at MVP.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
