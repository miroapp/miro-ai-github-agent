# Visual Template — Annotated Layout

The board is one frame composed of **4 vertical bands**. Every run produces the same shape so reviewers learn the layout once.

```
┌───────────────────────────────────────────────────────────────────────────────┐
│ Frame title:  "PR Review — <PR_TITLE>"                                        │
│                                                                               │
│ ┌───────────────────────────────────────────────────────────────────────────┐ │
│ │ BAND 1 — HEADER BANNER                                                    │ │
│ │ ┌─────────────────────────────────────────────────────────────────────┐   │ │
│ │ │ ▸ Eyebrow strip  (slate-900 fill, slate-50 text, small caps)        │   │ │
│ │ │   "PULL REQUEST REVIEW · SECURITY-CRITICAL"                         │   │ │
│ │ └─────────────────────────────────────────────────────────────────────┘   │ │
│ │ ▸ Title line (large, bold)                                                │ │
│ │   "PR #847 — Refactor Authentication to OAuth2 + JWT"                     │ │
│ │ ▸ Metadata line (small, muted)                                            │ │
│ │   "author: @dene  ·  branch: feat/oauth2 → main  ·  12 files  ·          │ │
│ │    +812 / −340 lines  ·  status: Ready for review"                       │ │
│ │ ▸ Topic-tag chip row                                                      │ │
│ │   [AUTH] [REFACTOR] [SECURITY] [OAUTH2] [RBAC] [MIGRATION]                │ │
│ └───────────────────────────────────────────────────────────────────────────┘ │
│                                                                               │
│ ┌───────────────────────────────────────────────────────────────────────────┐ │
│ │ BAND 2 — ARCHITECTURE BEFORE & AFTER                                      │ │
│ │ ┌──── SECTION TITLE STRIP (slate-100 fill) ──────────────────────────┐    │ │
│ │ │  Architecture — Before & After                                     │    │ │
│ │ │  Highlighted nodes show what changed in this PR                    │    │ │
│ │ └────────────────────────────────────────────────────────────────────┘    │ │
│ │                                                                           │ │
│ │   ┌────────────────────────┐         ┌────────────────────────┐           │ │
│ │   │ [❌ BEFORE — pill] red │         │ [✅ AFTER — pill] green│           │ │
│ │   │ ┌────────────────────┐ │         │ ┌────────────────────┐ │           │ │
│ │   │ │  BEFORE Mermaid    │ │         │ │  AFTER Mermaid     │ │           │ │
│ │   │ │  flowchart (red    │ │         │ │  flowchart (delta- │ │           │ │
│ │   │ │  palette)          │ │         │ │  marked nodes)     │ │           │ │
│ │   │ └────────────────────┘ │         │ └────────────────────┘ │           │ │
│ │   └────────────────────────┘         └────────────────────────┘           │ │
│ │                                                                           │ │
│ │   ┌── LEGEND BAR ──────────────────────────────────────────────────────┐  │ │
│ │   │ ▮ Removed/vulnerable  ▮ New  ▮ Modified  ▮ Unchanged               │  │ │
│ │   └────────────────────────────────────────────────────────────────────┘  │ │
│ └───────────────────────────────────────────────────────────────────────────┘ │
│                                                                               │
│ ┌───────────────────────────────────────────────────────────────────────────┐ │
│ │ BAND 3 — OWASP TOP 10                                                     │ │
│ │ ┌──── SECTION TITLE STRIP (red-50 fill) ────────────────────────────┐     │ │
│ │ │  OWASP Top 10 — Threats Addressed                                 │     │ │
│ │ │  Static analysis + manual review — severity-tagged findings       │     │ │
│ │ └───────────────────────────────────────────────────────────────────┘     │ │
│ │                                                                           │ │
│ │ ┌────────────────────────────────────────────────────────────────────┐    │ │
│ │ │ Table: OWASP ID | Threat | Where it lived | Mitigation | Severity  │    │ │
│ │ │ Sorted by severity desc; Severity column is fixed-set color-coded  │    │ │
│ │ └────────────────────────────────────────────────────────────────────┘    │ │
│ └───────────────────────────────────────────────────────────────────────────┘ │
│                                                                               │
│ ┌───────────────────────────────────────────────────────────────────────────┐ │
│ │ BAND 4 — CHANGE SUMMARY                                                   │ │
│ │ ┌──── SECTION TITLE STRIP (blue-50 fill) ───────────────────────────┐     │ │
│ │ │  Change Summary                                                   │     │ │
│ │ │  Files touched (with risk tag) — narrative of what changed        │     │ │
│ │ └───────────────────────────────────────────────────────────────────┘     │ │
│ │                                                                           │ │
│ │ ┌─────────────────────────┐    ┌──────────────────────────────┐           │ │
│ │ │ Files Changed table     │    │ Summary of Changes (doc)     │           │ │
│ │ │ Δ | Impact | File       │    │ # heading + emoji sections   │           │ │
│ │ │ Sorted by impact desc   │    │ skip empty sections          │           │ │
│ │ └─────────────────────────┘    └──────────────────────────────┘           │ │
│ └───────────────────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────────────────┘
```

## Widget-level breakdown

| Band | Widget | Tool | Notes |
|---|---|---|---|
| Header | Eyebrow strip | `shape_create` (rectangle) + `text_create` | Fill `#0F172A`, text `#F8FAFC`, small caps |
| Header | Title line | `text_create` | Large bold, color `#0F172A` |
| Header | Metadata line | `text_create` | Small, color `#64748B` (slate-500) |
| Header | Topic tag chip | `shape_create` (rounded rect) + `text_create` | One per tag; color per family (see `palette.md`) |
| Arch | Section title strip | `shape_create` (rectangle) + `text_create` | Fill `#F1F5F9`, text `#0F172A` |
| Arch | BEFORE pill | `shape_create` (rounded rect) + `text_create` | Fill `#FEE2E2`, stroke `#EF4444`, text `#991B1B` |
| Arch | AFTER pill | `shape_create` (rounded rect) + `text_create` | Fill `#DCFCE7`, stroke `#10B981`, text `#065F46` |
| Arch | BEFORE/AFTER diagram | `diagram_create_mermaid` | `flowchart TB`; always include classDef block |
| Arch | Legend swatch (×4) | `shape_create` (small square) | Per palette in `palette.md` |
| Arch | Legend label (×4) | `text_create` | Small, color `#0F172A` |
| OWASP | Section title strip | `shape_create` + `text_create` | Fill `#FEF2F2`, text `#7F1D1D` |
| OWASP | Threats table | `table_create` | Severity column is fixed-set (color-coded) |
| Change | Section title strip | `shape_create` + `text_create` | Fill `#EFF6FF`, text `#1E3A8A` |
| Change | Files Changed table | `table_create` | Impact column is fixed-set (color-coded) |
| Change | Summary doc | `doc_create` | Markdown; skip empty sections |

## Why 4 bands and not 3 or 5

3 bands lose the header banner — and the header is what makes the board work as a screenshot. 5 bands tempt the skill into manufacturing content (a "Tests" band, a "Performance" band) that's empty for most PRs. 4 is the smallest count that covers identity (header) + architecture (before/after) + risk (OWASP) + change narrative (summary).

## Sizing guidance

The frame's exact pixel size doesn't matter — Miro's layout tools handle it — but the **relative band heights** matter:

| Band | Rough height share |
|---|---|
| Header | 10% |
| Architecture (incl. legend) | 45% |
| OWASP | 20% |
| Change Summary | 25% |

The architecture band is the tallest because the diagrams need room to render at readable scale. If the OWASP table grows past 8 rows on a security-heavy PR, let the architecture band shrink to ~35% — never let any band fall below 10% or it becomes hard to scan.
