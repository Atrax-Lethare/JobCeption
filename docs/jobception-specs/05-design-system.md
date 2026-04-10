# Design System

- **Version:** 0.1
- **Status:** Draft
- **Owner:** Design
- **Last Updated:** 2026-04-09
- **Depends On:** —
- **Referenced By:** 05-ui-ux-spec.md, 06-frontend-spec.md

## Purpose
The single source of truth for visual design — tokens, typography, color, spacing, and components.

## Principles
- **Calm interface, loud signal.** Reputation, pledges, and constraints should pop; chrome should recede.
- **Information density without clutter.** Recruiters scan; candidates browse. Both need fast comprehension.
- **No dark patterns.** Pledge pop-ups have no sneaky defaults; salary disclosure is never hidden.

## Color Tokens
| Token | Light | Dark | Usage |
|---|---|---|---|
| `--bh-bg` | `#FFFFFF` | `#0B0F17` | Page background |
| `--bh-surface` | `#F5F7FA` | `#131822` | Cards |
| `--bh-text` | `#0B0F17` | `#E8ECF1` | Body |
| `--bh-text-muted` | `#5A6273` | `#9AA3B5` | Secondary |
| `--bh-primary` | `#1F3864` | `#6A8FD8` | Brand, primary CTA |
| `--bh-accent` | `#2E75B6` | `#79B5EC` | Links, highlights |
| `--bh-success` | `#1B7F3A` | `#4ED27B` | Pledge honored, success states |
| `--bh-warning` | `#C77700` | `#E5A14A` | Delay flags |
| `--bh-danger` | `#B0271E` | `#F26A60` | Pledge violation, errors |
| `--bh-aura` | `#7E57C2` | `#B388FF` | Aura points / XP |

## Typography
- **Primary face:** Inter.
- **Mono:** JetBrains Mono (sandbox).
- **Type scale:** 12, 14, 16, 18, 20, 24, 30, 36, 48, 60.
- **Line height:** 1.4 body, 1.2 headings.

## Spacing Scale
4, 8, 12, 16, 20, 24, 32, 40, 48, 64, 80.

## Radii
2, 4, 8, 12, 16, 24, 9999 (pill).

## Elevation
Soft shadows only; never harder than `0 6px 24px rgba(0,0,0,.08)`.

## Component Library
Built on top of Radix UI primitives with Tailwind theming. Inventory:

- Button (primary / secondary / ghost / danger)
- Input, Textarea, Select, MultiSelect, Combobox
- Checkbox, Radio, Switch, Slider
- Dialog (incl. Pledge Pop-up variant)
- Drawer, Sheet
- Tabs, Accordion
- Card, ListItem
- Avatar, AvatarFrame, Badge
- Tag, Chip
- Toast, Banner
- Tooltip, Popover
- ProgressBar, ProgressIndicator (multi-stage)
- Skeleton, Spinner
- DataTable
- Markdown renderer (sanitized)
- CodeEditor (Monaco wrapper)

## Iconography
- Lucide as the base set.
- Custom icons for Aura, Bounty, Pledge.

## Motion
- Default: 150–200 ms ease-out.
- Pledge Pop-up confirm: deliberate 300 ms scale + slight haptic on touch devices.
- Reduce Motion respected globally.

## Open Questions
- Bespoke illustration set or stock illustration partner?
- Mascot or no mascot?

## Change Log
- 2026-04-09 — v0.1 — Initial draft.
