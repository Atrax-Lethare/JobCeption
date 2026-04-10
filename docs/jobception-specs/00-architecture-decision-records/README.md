# Architecture Decision Records

This directory holds one Markdown file per significant architectural decision in JobCeption.

## When to Write an ADR
Write an ADR when a decision:
- Is hard to reverse (e.g., choice of database, escrow provider, sandbox runtime).
- Constrains future work (e.g., mandating GraphQL over REST).
- Is contested or non-obvious.
- Has legal, security, or compliance implications.

## Naming
`NNNN-short-kebab-title.md` where `NNNN` is a zero-padded sequence number, e.g. `0001-postgres-as-primary-db.md`.

## Template
```markdown
# ADR NNNN — [Title]

- **Status:** Proposed | Accepted | Deprecated | Superseded by ADR-XXXX
- **Date:** YYYY-MM-DD
- **Deciders:** [names / roles]
- **Related Specs:** [list]

## Context
What is the issue we're seeing that motivates this decision?

## Decision
What is the change we're making?

## Consequences
What becomes easier? What becomes harder? What are the trade-offs?

## Alternatives Considered
What else was on the table and why was it rejected?
```

## Initial ADRs to Author
The following decisions are flagged in the spec set and should be captured as ADRs before MVP launch:
- 0001 — Primary database choice (PostgreSQL vs. alternatives)
- 0002 — Bounty sandbox runtime (Firecracker vs. gVisor vs. plain Docker)
- 0003 — Escrow provider (Stripe Connect vs. RazorpayX vs. custom)
- 0004 — Auth provider (build vs. WorkOS / Clerk / Auth0)
- 0005 — Pledge legal binding model (jurisdiction-specific)
- 0006 — XP & dilution ratio formula (and how it can be re-tuned without resetting accounts)
- 0007 — Review moderation policy (pre- vs. post-publication)
