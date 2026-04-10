# Testing Strategy

- **Version:** 0.1
- **Status:** Draft
- **Owner:** QA / Tech Lead
- **Last Updated:** 2026-04-09
- **Depends On:** 01-non-functional-requirements.md
- **Referenced By:** —

## Purpose
Define what we test, at what layer, with what tools, and the coverage bar.

## Test Pyramid
- **Unit (most):** business logic, pure functions, validators.
- **Integration:** services + DB + external mocks.
- **Contract:** API contract tests against the OpenAPI schema.
- **E2E (fewest):** critical user journeys via Playwright.
- **Load:** k6 against staging on a schedule.
- **Chaos:** quarterly game days against staging (kill workers, pause queues).

## Coverage Targets
- Backend: ≥ 80% lines, ≥ 70% branches.
- Frontend: ≥ 70% lines.
- Critical modules (pledge, escrow, reputation): ≥ 95% lines, branch coverage required.

## Tools
- Backend unit/integration: Jest/Vitest (TS) or Go test.
- API contract: Schemathesis or Dredd.
- Frontend: Vitest + React Testing Library.
- E2E: Playwright.
- Load: k6.

## Critical Test Suites
- **Pledge state machine:** every transition, every invariant, every edge case.
- **Bounty publish immutability:** verify no field can change post-publish.
- **Reputation event ledger:** append-only enforcement.
- **Sandbox escape attempts:** known CVE patterns, fuzzing.
- **Authorization matrix:** every role × every endpoint.

## Test Data
- Anonymized snapshots for staging.
- Factories for unit tests.
- Seed scripts for E2E.

## Release Gates
- All tests green on `main`.
- Coverage delta non-negative.
- No critical SAST findings.

## Open Questions
- Mutation testing for pledge module.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
