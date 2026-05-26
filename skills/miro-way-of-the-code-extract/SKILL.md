---
name: miro-way-of-the-code-extract
description: Use when the user wants to start an architectural planning session on Miro — extracts the current codebase architecture into two side-by-side frames ("Current Architecture" and "Future Architecture", both pre-populated with the same diagrams) so the user can edit the Future frame to design changes. Pairs with miro-way-of-the-code-implement, which later turns the user's edits into a PR.
---

# Way of the Code — Extract

Set up an architectural planning canvas on a Miro board. Render the current architecture (systems, modules, user flows) identically on two large side-by-side frames titled **Current Architecture** and **Future Architecture**. The user modifies the diagrams in the Future Architecture frame to design proposed changes; the companion skill `miro-way-of-the-code-implement` then reads the delta and produces a PR.

The user optionally supplies a Miro board URL. If none is given, the skill creates a fresh board via the Miro MCP `board_create` tool.

## Workflow

### 1. Analyze the codebase

Reuse the analysis steps from `miro-code-to-board-architecture`:

- §1 Inventory the repo (identify systems)
- §2 Map modules and dependencies
- §3 Trace user flows
- §4 Build Mermaid diagrams: system map (§4a), per-system module diagrams (§4b), user flows (§4c)

Skip the security analysis (§5) and code-quality analysis (§6) — they're out of scope for an architectural planning canvas.

### 2. Resolve the Miro board

If the user supplied a board URL, capture it as `BOARD_URL` and announce `Using board: <BOARD_URL>` in chat. Otherwise create a fresh board with `board_create`. Name it `WotC: <repo-name>` (WotC = Way of the Code). Announce in chat:

> Created Miro board: <BOARD_URL>

### 3. Create the two frames

Create **two large frames side-by-side** with `layout_create`, in this left-to-right order:

| Position | Frame title | Purpose |
|---|---|---|
| Left | `Current Architecture` | The as-is state — do not modify after creation |
| Right | `Future Architecture` | Initially identical to the left frame; the user edits this to design changes |

Size each frame wide enough to hold the system map at a readable scale (it's the largest single artifact). Both frames must be the **same dimensions** so the side-by-side comparison feels deliberate. If `layout_create`'s schema isn't familiar, call `layout_get_dsl` first to inspect a template.

Capture the resulting IDs as `CURRENT_FRAME_ID` and `FUTURE_FRAME_ID`.

### 4. Populate both frames identically

For **each** of the two frames, place the same set of items in the same vertical order. Pairing across frames relies on the **H1 line** of each section-overview doc, so the H1 must stay identical across frames; the doc body may diverge (e.g. the user might add planning notes in the Future frame).

Each section is a pair: a **section-overview doc** (with H1 + body describing the section) followed by a **Mermaid diagram** that visualizes the same content. The doc carries the prose, the diagram carries the shape — both are needed.

Inside each frame, top-to-bottom:

1. **System Map overview doc** (`doc_create`) — see template below
2. The **§4a system map** Mermaid diagram (`diagram_create_mermaid`)
3. For each system identified in §1, in alphabetical order by system name:
   1. **Module overview doc** (`doc_create`) — see template below
   2. The §4b module diagram for that system (`diagram_create_mermaid`)
4. For each principal user flow from §3, in the order they were captured:
   1. **Flow overview doc** (`doc_create`) — see template below
   2. The §4c sequence diagram (`diagram_create_mermaid`)

#### Doc templates

Use these as the body of each section-overview doc. Keep each one concise — a reviewer should be able to skim it in 30 seconds. Omit a section heading (e.g. "External dependencies") when it has no content rather than leaving it empty.

**System Map overview:**

```markdown
# System Map

<one paragraph describing what the codebase is at a high level — monorepo vs single app, the user-facing surface, the dominant runtime>

## Systems

- **<system-name>** (<type>, <runtime>) — <purpose>; <public surface in one phrase>
- **<system-name>** (<type>, <runtime>) — …

## Cross-system surfaces

- <surface> (e.g. `HTTP /api/v1`) — <who calls who>
```

**Module overview** (one per system):

```markdown
# Module Structure: <system-name>

<one paragraph describing the system's role in the codebase, its entry point, and how the modules below fit together>

## Modules

- **<module-name>** — <responsibility>; depends on <internal modules>; exports/surfaces <Z>
- **<module-name>** — …

## External dependencies

- <package or service> — <what it's used for>
```

**Flow overview** (one per flow):

```markdown
# Flow: <flow-name>

<one paragraph describing the flow from external trigger to terminal effect>

## Actors

- <actor> — <role>

## Steps

1. <step-1 description, naming the module/handler it lives in>
2. <step-2 description>
…

## Datastores / external services touched

- <store> — <read/write/both>
```

**Tool reference for this step:**

| Tool | Used for |
|---|---|
| `layout_create` | Create the two frames; set frame titles as part of the payload |
| `layout_get_dsl` | Inspect the layout DSL schema if needed |
| `diagram_create_mermaid` | Render every Mermaid diagram twice — once per frame |
| `doc_create` | Each section-overview doc (H1 + body per the templates above) |
| `board_list_items` | Verify the H1 lines across frames match after population |

After populating both frames, verify with `board_list_items` that every section's H1 line appearing in the Current frame also appears verbatim in the Future frame. If any mismatch exists, fix it before reporting.

### 5. Add a guidance note above the frames

Create one more `doc_create` positioned **above** the two frames (not inside either), with this content:

```markdown
# How to use this canvas

This board has two frames: **Current Architecture** (left) and **Future Architecture** (right).

Both frames currently show the same diagrams — the as-is state of the codebase.

To design changes:

1. Leave the **Current Architecture** frame untouched.
2. In the **Future Architecture** frame, modify the Mermaid diagrams to reflect the architecture you want — add nodes, remove edges, rename components, restructure flows, etc.
3. Keep diagram titles (section headers) stable. The implement skill pairs Current ↔ Future diagrams by their titles.
4. When you're done, run `miro-way-of-the-code-implement` with this board's URL to generate a PR that turns the deltas into code changes plus an implementation plan.
```

### 6. Output

Report in chat:

1. `BOARD_URL`
2. The two frame IDs (so the user can deep-link to either: `<BOARD_URL>?moveToWidget=<FRAME_ID>`)
3. A summary: `<N> systems, <F> user flows, <D> diagrams placed in each frame`
4. A reminder: `"Edit the Future Architecture frame; do not edit Current Architecture."`

## Background

### Pairing convention

The implement skill matches sections across frames by **the H1 line** of each section-overview doc — not the full doc body. Don't rename the H1, don't insert extra H1-headed docs, don't reorder unless you can guarantee the same reorder in both frames. The implement skill handles three diff cases:

- H1 exists in both frames → modified diagram, structural diff applied to the paired Mermaid
- H1 only in Future → additive change (new system / module / flow)
- H1 only in Current → removal

All three are valid. The constraint is that the H1 line stays stable across edits; the doc body can diverge freely (the user may add planning notes in the Future doc, and that's fine).

### Why two identical frames

Architectural design is easier when the as-is and to-be are visible side by side under the same conventions. A single before/after diagram inside one frame is too cramped; two full frames give the user room to think and to rearrange.

### Layout reference

```
┌──────────────────────────────────┐
│  How to use this canvas (doc)    │
└──────────────────────────────────┘

┌────────────────┐  ┌────────────────┐
│  Current       │  │  Future        │
│  Architecture  │  │  Architecture  │
│                │  │                │
│  System Map    │  │  System Map    │
│  [diagram]     │  │  [diagram]     │
│                │  │                │
│  Module ...    │  │  Module ...    │
│  [diagram]     │  │  [diagram]     │
│                │  │                │
│  Flow: ...     │  │  Flow: ...     │
│  [diagram]     │  │  [diagram]     │
│                │  │                │
│  (do not edit) │  │  (edit me)     │
└────────────────┘  └────────────────┘
```
