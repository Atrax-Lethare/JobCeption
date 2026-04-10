# Accessibility Spec

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Design
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** —

## Purpose
Define the accessibility floor for JobCeption.

## Target
WCAG 2.2 AA on every candidate-facing surface. WCAG 2.2 AAA where practical.

## Requirements
- **Keyboard navigation:** every interactive element reachable and operable via keyboard. Visible focus ring on all focusable elements.
- **Screen reader:** semantic HTML, ARIA only when semantic HTML insufficient. Every form field has a programmatic label.
- **Color contrast:** body text ≥ 4.5:1; large text ≥ 3:1; UI components ≥ 3:1.
- **Motion:** `prefers-reduced-motion` respected globally.
- **Resize:** content reflows at 200% zoom without loss of function.
- **Captions:** required for any video content.
- **Alt text:** required for all meaningful images; decorative images use `alt=""`.
- **Forms:** clear error messages tied to fields via `aria-describedby`; errors announced.

## Sandbox-Specific
- Editor must support screen reader mode (Monaco supports this).
- Anti-Cheat features must not penalize assistive tech use (e.g., dictation, screen readers).

## Testing
- Automated: axe-core in CI on every page.
- Manual: keyboard-only walkthrough each release.
- External audit annually.

## Open Questions
- Can the Sandbox meet AA for blind candidates given the inherently visual nature of some bounty types?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
