# Palette Reference

All colors are hex codes drawn from the Tailwind palette so they render consistently across screens, screenshots, and projected slides.

## Core delta palette (shared with `miro-code-review`)

| Semantic | Fill | Stroke | Text | Mermaid classDef name |
|---|---|---|---|---|
| Added / new | `#DCFCE7` (green-100) | `#10B981` (emerald-500) | `#065F46` (green-800) | `new` |
| Modified | `#FEF3C7` (amber-100) | `#F59E0B` (amber-500) | `#92400E` (amber-800) | `modified` |
| Removed / vulnerable | `#FEE2E2` (red-100) | `#EF4444` (red-500) | `#991B1B` (red-800) | `removed` |
| Unchanged | `#DBEAFE` (blue-100) | `#3B82F6` (blue-500) | `#1E3A8A` (blue-900) | `unchanged` |

Use these on:

- Every node of every BEFORE/AFTER diagram
- Every Severity / Impact cell of every table (stroke colors only)
- Every legend swatch in the architecture band

## Section title strip accents

The four bands use distinct strip colors so a viewer scrolling at any zoom level immediately knows which band they're in.

| Band | Strip fill | Strip text |
|---|---|---|
| Header banner | `#0F172A` (slate-900) | `#F8FAFC` (slate-50) |
| Architecture | `#F1F5F9` (slate-100) | `#0F172A` (slate-900) |
| OWASP | `#FEF2F2` (red-50) | `#7F1D1D` (red-900) |
| Change Summary | `#EFF6FF` (blue-50) | `#1E3A8A` (blue-900) |

The header banner is the only "dark" strip — it visually anchors the top of the board.

## BEFORE / AFTER pill labels

These sit directly above each diagram. Pills are rounded rectangles (corner radius ≈ height/2) so they read as labels, not panels.

| Pill | Fill | Stroke | Text | Sample label |
|---|---|---|---|---|
| `BEFORE` | `#FEE2E2` | `#EF4444` | `#991B1B` | `❌ BEFORE — Insecure Monolithic Auth` |
| `AFTER` | `#DCFCE7` | `#10B981` | `#065F46` | `✅ AFTER — Hardened OAuth2 / JWT Flow` |

The pill text **must end with a one-line subtitle** that names the architectural shift, not just "BEFORE" / "AFTER". Examples:

- `BEFORE — N+1 queries on dashboard load` → `AFTER — Single batched query with cache`
- `BEFORE — Inline SQL in handlers` → `AFTER — Repository pattern + parameterized queries`
- `BEFORE — Polling on 5s interval` → `AFTER — Server-sent events`

## Topic-tag chip families

Chips on the header banner. Pick the family by tag content; one fill per family.

### Security family

For: `SECURITY`, `AUTH`, `AUTHN`, `AUTHZ`, `RBAC`, `OAUTH2`, `JWT`, `CRYPTO`, `SECRETS`

| Property | Value |
|---|---|
| Fill | `#FECACA` (red-200) |
| Text | `#7F1D1D` (red-900) |
| Stroke | none |

### Risk family

For: `BREAKING`, `MIGRATION`, `DEPRECATION`

| Property | Value |
|---|---|
| Fill | `#FED7AA` (orange-200) |
| Text | `#9A3412` (orange-900) |
| Stroke | none |

### Domain family

For: `UI`, `API`, `DATA`, `INFRA`, `DOCS`, `BILLING`, `SEARCH`, `NOTIFICATIONS`, `EXPORT`

| Property | Value |
|---|---|
| Fill | `#DBEAFE` (blue-100) |
| Text | `#1E3A8A` (blue-900) |
| Stroke | none |

### Type family

For: `REFACTOR`, `FEATURE`, `BUGFIX`, `CHORE`, `PERF`, `A11Y`, `I18N`

| Property | Value |
|---|---|
| Fill | `#E9D5FF` (purple-200) |
| Text | `#581C87` (purple-900) |
| Stroke | none |

## Banner metadata text colors

| Element | Color |
|---|---|
| Title line | `#0F172A` (slate-900) |
| Metadata line | `#64748B` (slate-500) |
| Eyebrow strip text | `#F8FAFC` (slate-50) |
| Eyebrow strip fill | `#0F172A` (slate-900) |

## Table column color codes (fixed-set cells)

Applied to the **Severity** column of the OWASP table and the **Impact** column of the Files Changed table.

| Value | Stroke color | Fill |
|---|---|---|
| High | `#EF4444` (red-500) | `#FEE2E2` (red-100) |
| Medium | `#F59E0B` (amber-500) | `#FEF3C7` (amber-100) |
| Low | `#10B981` (emerald-500) | `#DCFCE7` (green-100) |

## What not to use

- ❌ Pure-saturation colors (`#FF0000`, `#00FF00`) — they vibrate on projectors
- ❌ Theme colors picked per-PR — kills cross-PR consistency
- ❌ Stroke-only nodes with no fill — read as "ghosted/disabled" on Miro
- ❌ More than 6 topic-tag chips — wraps the header onto two lines

## Why these specific hexes

The Tailwind 100/500/800 stops give a consistent visual weight across families: 100 fills are always light enough to put dark text on, 500 strokes are saturated enough to read at low zoom, 800–900 text always passes WCAG AA on the matching 100 fill. Mixing custom hexes erodes that guarantee.
