# JobCeption Prototype — Working Agreement

This is a 15-hour hackathon prototype build. Specs live in docs/jobception-specs/.

## What we're building
A working end-to-end BOUNTY system (creation, marketplace, participation, sandbox via Judge0 CE public, automatic evaluation, submissions, leaderboard) plus UI-only mocks of the rest of the product.

## Stack (locked, all free tier, no credit cards)
- Frontend: Next.js 14 App Router, TypeScript strict, Tailwind, shadcn/ui, Monaco editor → Vercel
- Backend: FastAPI (Python 3.11), async SQLAlchemy 2.0, Pydantic v2 → Render free tier
- DB & Auth: Supabase (Postgres + Auth) free tier
- Sandbox: Judge0 CE public instance at https://ce.judge0.com (no auth, no key)

## Rules
- Read the relevant spec file in docs/jobception-specs/ before implementing any feature. Cite it.
- Prefer small commits. One feature per commit. Use conventional commit messages.
- TypeScript strict on frontend. Pydantic models for all FastAPI I/O.
- NEVER use hosted LLM APIs in this prototype (Gemini/Anthropic/OpenAI) — not needed.
- If a feature is on the "UI-only mocks" list below, build the UI only. Do not wire it to a backend.
- Stop and ask if a task would take more than 30 minutes or touches more than 6 files.
- Use the REST Client extension format (.http files) for backend testing examples when you show me commands.

## Built for real
Auth, roles (candidate/bounty_creator), bounty CRUD for creators, marketplace,
participation, sandbox with Monaco + Judge0 CE public, automatic evaluation on submit,
performance report, submission history, leaderboard.

## UI-only mocks
Landing page, Pledge Pop-up, hiring pipeline, dynamic profile (with real submissions),
connections feed, reviews, admin stub.

## Not in scope
Real pledges, reputation ledger, escrow, KYC, notifications, plagiarism detection,
behavioral biometrics, recommendation engine, SonarQube, multiple bounty types.

## Key constraints
- Judge0 URL is https://ce.judge0.com, token is empty/not sent (public instance, no auth)
- Render free tier spins down after 15 min — accept this; document it
- Supabase free tier limits are fine for prototype scale

## Preferred patterns
- Use shadcn/ui components via CLI (`npx shadcn@latest add ...`)
- Use @supabase/ssr for Next.js auth
- FastAPI routes grouped by resource (routers/bounties.py, routers/submissions.py)
- Database models in backend/app/models/
- All HTTP calls from frontend go through a typed client in apps/web/lib/api.ts