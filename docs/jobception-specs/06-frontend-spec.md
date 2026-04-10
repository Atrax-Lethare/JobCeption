# Frontend Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Frontend
- **Last Updated:** 2026-04-09
- **Depends On:** 03-tech-stack.md, 05-design-system.md, 04-authorization-spec.md
- **Referenced By:** —

## Purpose
Routing, state management, data fetching, rendering strategy, and code organization for the web client.

## App Structure
Single Next.js app split into route groups:
- `(public)` — marketing, public marketplace, public profiles
- `(candidate)` — authenticated candidate surfaces
- `(recruiter)` — authenticated recruiter surfaces
- `(admin)` — internal staff
- `(auth)` — sign-in / sign-up flows

Shared `app/_components/` for cross-group components; per-group `_components/` for scoped ones.

## Rendering Strategy
- **SSR** for SEO-relevant pages (marketplace, public bounty detail, company profiles).
- **CSR** for app-shell heavy pages (sandbox, dashboards).
- **ISR** for slowly-changing public pages.
- **Streaming** with React Suspense for above-the-fold-first pages.

## State Management
- **Server state:** TanStack Query. One query key per resource; invalidate on mutation.
- **UI state:** Zustand stores per feature. No global megastore.
- **Form state:** React Hook Form + Zod schemas (shared between client and server validation).

## Data Fetching
- All API calls go through a typed client generated from the OpenAPI schema.
- Mutation responses optimistically update caches where safe.
- Idempotency keys auto-attached to pledge and escrow mutations.

## Routing
- File-based via Next.js App Router.
- Auth gating in route group `layout.tsx` + middleware for unauthenticated redirects.
- Permission gating via a `<RequirePermission>` boundary component.

## Sandbox Frontend
- Monaco editor wrapper.
- WebSocket client wraps reconnection + session resume logic.
- Exposes Anti-Cheat hooks (visibility, paste) at the document level.
- Strict CSP — no `eval`, no inline scripts.

## Performance
- LCP budget 2.5 s on 4G.
- Hydration on-demand for non-critical components.
- Image optimization via Next.js `<Image>`.
- Font subsetting for Inter.

## Internationalization
- All strings via `next-intl`. English-first; locale folder structure ready.

## Error Boundaries
- Per route group. See `06-error-handling-spec.md`.

## Testing
- Component tests with Vitest + React Testing Library.
- E2E with Playwright.

## Open Questions
- Use Server Actions for mutations or stick with the API client?
- Client-side feature flag SDK choice.

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
