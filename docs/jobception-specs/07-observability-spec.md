# Observability Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** DevOps
- **Last Updated:** 2026-04-09
- **Depends On:** 01-non-functional-requirements.md
- **Referenced By:** —

## Purpose
Logging, metrics, tracing, dashboards, and alerting.

## Logs
- Structured JSON.
- Required fields: `ts`, `level`, `service`, `request_id`, `user_id` (when available), `event`, `msg`.
- PII scrubbed at the logger.
- Log levels: `debug`, `info`, `warn`, `error`. No `info` spam in hot loops.
- Retention: 30 days hot, 1 year cold for audit logs.

## Metrics
- Prometheus + Grafana.
- RED metrics on every endpoint (Rate, Errors, Duration).
- USE on infra (Utilization, Saturation, Errors).
- Business metrics: bounties published, submissions completed, pledges issued, ghost rate.

## Tracing
- OpenTelemetry. Every request gets a trace ID propagated end-to-end (browser → API → services → DB).
- Sample rate: 100% for pledge writes, 10% otherwise.

## Dashboards
- Per service: latency, error rate, saturation.
- Per business KPI: time-to-hire, ghost rate, pledge honor rate.
- Sandbox: pool depth, cold-start latency, escapes (target 0).

## Alerts
- Pager-grade: p95 latency > SLO for 5 min, error rate > 1% for 5 min, pledge service down, sandbox escape.
- Email/Slack: dependency degraded, scheduled-job failures, DLQ depth > threshold.

## Audit Log
- Immutable, append-only.
- Mirrored to a separate AWS account for tamper resistance.

## Open Questions
- Trace sampling cost vs. visibility trade-off.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
