---
name: miro-code-to-board-architecture
description: Use when the user wants a comprehensive architectural overview of an entire codebase rendered on a Miro board — system inventory, module dependencies, user flows, security analysis (OWASP), and per-module code-quality findings, organized as frames.
---

# Code-to-Board Architecture

Audit an entire repository and turn it into a structured architectural review on a Miro board: discovered systems, their dependencies, user flows, security-critical paths with OWASP analysis, and per-module code-quality findings. The output is a frame-per-topic board the user can pan through.

The user optionally supplies a Miro board URL. If none is given, the skill creates a fresh board via the Miro MCP `board_create` tool (see §7).

## Workflow

### 1. Inventory the repo

Walk the working directory and identify the discrete **systems** in the repo. A system is anything with its own entry point and runtime: a frontend app, a backend service, a CLI tool, a worker, a library/SDK, an infra module (Terraform, Helm), etc.

**Signals to identify systems:**

- Top-level manifests: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `pom.xml`, `Gemfile`, `composer.json`
- Containerization: `Dockerfile`, `docker-compose.yml`, `Procfile`
- Build configs at non-root paths: `webpack.config.*`, `vite.config.*`, framework-specific configs
- Workspace declarations: `pnpm-workspace.yaml`, `lerna.json`, `nx.json`, `turbo.json`, Cargo / Go workspaces
- Infra: `terraform/`, `k8s/`, `helm/`, `infra/`
- Conventional layout: `apps/`, `services/`, `packages/`, `cmd/`

For each system, classify:

| Aspect | Examples |
|---|---|
| Type | frontend SPA, frontend SSR, backend HTTP, backend RPC, async worker, CLI, library, infra module |
| Runtime | Node, Python, Go, Rust, JVM, etc. |
| Entry point | path to main file / route table / framework bootstrap |
| Public surface | HTTP routes, gRPC methods, CLI commands, exported library API |

If the repo is a single application (not a monorepo), treat it as one system and skip straight to module-level analysis in §2.

### 2. Map modules and dependencies

For each system identified in §1:

**Internal modules:**

- Read the system's manifest to enumerate external dependencies
- Build an import graph for the system's own source (parse imports/requires/uses)
- Group source files into modules using the directory structure (or framework conventions: `controllers/`, `services/`, `lib/`, `internal/<pkg>/`, etc.)

**Cross-system dependencies:**

- Detect when one system depends on another (workspace imports, shared internal packages, HTTP/RPC clients pointing at sibling services)
- Record the direction and the surface used (HTTP path, gRPC method name, library entry point)

### 3. Trace user flows

For each system that has a user-facing surface, trace the principal flows:

- **Frontends** — routes / pages, the calls each page makes, the components it renders
- **Backends** — routes / RPCs, the auth boundary, the downstream services / datastores each handler touches
- **CLIs / workers** — commands or trigger events, the work each performs

A flow is a path from external input to terminal effect (DB write, response to user, queue publish). Capture flows that span multiple systems separately — they are the most important for the cross-system view in §4.

### 4. Build the architectural diagrams (Mermaid)

Codify the inventory into three diagram families. Generate them all as Mermaid; they're created on the board in §8 via the Miro MCP `diagram_create_mermaid` tool.

**4a. System map (one diagram, top-level).**

A flowchart showing every system as a node, with edges for cross-system dependencies. Annotate edges with the surface (`HTTP /api/users`, `gRPC Auth.Verify`, `imports @org/lib-x`). Include external dependencies (DBs, queues, third-party APIs) as differently-styled nodes.

**4b. Module structure (one diagram per system).**

A class or component diagram per system, showing internal modules and their dependencies. Group external deps into a separate cluster. For very small systems (≤ 3 modules), skip the per-system diagram and let the system map carry it.

**4c. User flows (one diagram per principal flow).**

A sequence diagram per flow showing actors → systems → datastores. For purely intra-system flows where the sequence is degenerate, a flowchart is acceptable.

### 5. Security analysis

Identify flows where **data security** or **system integrity** is critical. Heuristics for which flows qualify:

- Authentication / authorization boundaries (login, token issuance, session management)
- User input that hits a DB, a shell, an HTTP fetch, or a templating engine
- File upload / download
- Payments, billing, PII handling
- Multi-tenant boundaries (one tenant's data must not leak to another)
- Webhook receivers / OAuth callbacks
- Admin endpoints
- Anything touching crypto keys, secrets, or session material

For each critical flow, produce four artifacts:

**5a. OWASP investigation.** Walk through the OWASP Top 10 and note which categories are relevant to this flow. Skip categories that don't apply.

- A01 Broken Access Control
- A02 Cryptographic Failures
- A03 Injection
- A04 Insecure Design
- A05 Security Misconfiguration
- A06 Vulnerable & Outdated Components
- A07 Identification & Authentication Failures
- A08 Software & Data Integrity Failures
- A09 Security Logging & Monitoring Failures
- A10 SSRF

**5b. Flow diagram.** A sequence diagram showing the request path with trust boundaries marked. Annotate where authentication, authorization, validation, and sanitization happen — or are missing.

**5c. Flow description document.** A short markdown doc covering:

- Purpose of the flow
- Trust boundary (who can call it, what credentials are required)
- Inputs and how they are validated
- Outputs and how they're exposed
- Datastores touched
- Known gaps

**5d. Findings table.** A table of identified issues for this flow:

| Column | Notes |
|---|---|
| Issue | Short description of what's wrong |
| OWASP Category | A01–A10 reference |
| Location | `file:line-line` (clickable when a remote URL is known) |
| Impact | High / Medium / Low (color-coded fixed-set column) |
| Recommendation | Short fix description |

Order rows by impact descending (High first).

### 6. Code-quality analysis

For each module identified in §2, evaluate:

**Quality signals:**

- Cyclomatic complexity (heuristic: branches per function; flag > 10)
- Function length (flag > 50 lines)
- File length (flag > 400 lines)
- Duplication (repeated blocks across the module)
- Naming consistency
- Comment-to-code ratio (extremes in either direction are a smell)
- Coupling (number of imports from other modules)

**Test coverage:**

- Look for colocated `__tests__/`, `*_test.go`, `*.test.ts`, `tests/`, etc.
- Read coverage reports if present (`coverage/`, `.coverage`, `coverage.out`, etc.) and capture the numbers
- If no tests exist for a module, mark it explicitly

**System quality:**

- Error handling (silent failures, broad catches, swallowed errors)
- Logging discipline (over-logging, missing logs at boundaries)
- Type safety (`@ts-ignore`, `any`, unchecked casts, `interface{}` abuse)
- Documentation (public APIs without docstrings)

Produce a **per-module findings table**:

| Column | Notes |
|---|---|
| Problem | What's wrong |
| Location | `file:line-line` (clickable when remote known) |
| Category | Complexity / Duplication / Tests / Error Handling / Types / Docs |
| Impact | High / Medium / Low (color-coded) |
| Effort | Quick / Moderate / Heavy |

Order rows by impact descending.

### 7. Resolve the Miro board

If the user supplied a board URL, capture it as `BOARD_URL` and announce `Using board: <BOARD_URL>` in chat. Otherwise create a fresh board with the Miro MCP `board_create` tool *before* §8 — every downstream step assumes the board exists.

**Naming the new board:**

- Single-system repo → `Architecture: <repo-name>`
- Monorepo → `Architecture: <repo-name> (N systems)`

Announce in chat:

> Created Miro board: <BOARD_URL>

### 8. Compose the board

The board is laid out as a **sequence of frames** in a single horizontal row, each frame holding a logical chunk of the analysis. Create frames with the Miro MCP `layout_create` tool — pass the frame's title as part of the layout payload (no separate text widget needed for titles). If you're unsure about the layout DSL shape, call `layout_get_dsl` first to inspect a template.

**Miro MCP tools used in this step:**

| Tool | Used for |
|---|---|
| `layout_create` | Create each frame; set its title as part of the layout payload |
| `layout_get_dsl` | Inspect the layout DSL schema if needed |
| `diagram_create_mermaid` | Render every Mermaid diagram from §4 and §5b |
| `doc_create` | Create every summary / overview / flow-description document |
| `table_create` | Create the §5d security findings and §6 quality findings tables |
| `code_widget_create` | Embed code snippets for High-impact module findings |
| `board_list_items` | Re-read item IDs / positions if you need to reflow after creation |

Frame order (left-to-right):

| # | Frame | Contents | Size |
|---|---|---|---|
| 1 | Architecture Overview | §4a system map + summary doc | Large |
| 2 | System Structure & Flows | §4b module diagrams + §4c user-flow diagrams, grouped by system | Large |
| 3..(2+S) | Security flows | One frame per critical flow from §5 | Standard |
| (3+S)..end | Code-quality per module | One frame per module from §6 (skip clean modules) | Standard |

Inside each frame, stack contents top-to-bottom.

#### Frame 1 — Architecture Overview

- Frame title (on `layout_create`): `Architecture Overview`
- The **system map** Mermaid diagram from §4a — `diagram_create_mermaid`
- A summary document — `doc_create` — listing:
  - Total systems and their types
  - Principal cross-system dependencies
  - Datastores and external services in use
  - Anything surprising or non-obvious about the topology

Size the frame wide enough that the system map renders at readable scale without zooming.

#### Frame 2 — System Structure & Flows

- Frame title (on `layout_create`): `System Structure & Flows`

Per system, in a vertical stack inside the frame:

- A **module overview doc** (`doc_create`) — H1 `Module Structure: <system-name>`, then a one-paragraph role description, a `## Modules` section listing each module with responsibility + dependencies + exports, and a `## External dependencies` section if non-empty. Keep it skim-readable.
- The §4b module diagram for that system — `diagram_create_mermaid`
- For each user-flow owned by this system:
  - A **flow overview doc** (`doc_create`) — H1 `Flow: <flow-name>`, then a one-paragraph trigger-to-effect description, a `## Steps` list (numbered, naming the module/handler each step lives in), and `## Datastores touched` if non-empty.
  - The §4c sequence diagram — `diagram_create_mermaid`

After all per-system stacks, place **cross-system flows** as a final section — these span boundaries and most often surface architectural issues. Use the same flow-overview doc shape for each.

Never emit a section as an H1-only doc with no body — the diagram visualizes the shape, the doc carries the prose. Both are needed.

#### Frames 3..(2+S) — Security flows (one frame per critical flow)

For each flow identified in §5:

- Frame title (on `layout_create`): flow name + risk level, e.g. `Login flow — High risk`
- Flow sequence diagram from §5b — `diagram_create_mermaid`
- Flow description document from §5c — `doc_create`
- Findings table from §5d — `table_create`, with the `Impact` column as a fixed-set column (High = red, Medium = orange, Low = green); rows sorted by impact

If §5 found no security-critical flows, skip these frames entirely and note it once in chat: `"No security-critical flows identified — skipping per-flow frames."`

#### Frames (3+S)..end — Code-quality per module

For each module from §6 that has at least one finding (or a notable test-coverage gap):

- Frame title (on `layout_create`): `<module-name> — <parent-system>`
- A short overview document — `doc_create` — covering the quality signals and test-coverage notes
- The findings table — `table_create`, `Impact` as a fixed-set color-coded column, rows sorted by impact
- For each **High-impact** issue, embed a **code widget** — `code_widget_create` — containing the problematic snippet. Caption with `file:line-line`; body is the offending code (or a minimal diff if the issue is a relationship between two snippets).

Skip modules that have no findings and no coverage concerns — an empty frame is noise.

### 9. Output

After composition, report in chat:

1. `BOARD_URL`
2. Summary: `<N> systems, <M> modules, <S> security flows analyzed, <F> frames created`
3. **Top architectural concerns** — 3–5 bullets
4. **Top security findings** — High-impact items only, with the frame they live in
5. **Top quality findings** — High-impact items only, with the frame they live in

## Background

### Why frames

This skill produces a lot of content. Frames let a viewer pan through the analysis at the granularity that matters to them — a developer wants the system map and one module; a security reviewer wants the security flows; a PM wants the overview. Each frame is self-contained.

### Don't manufacture content

If a system has no internal modules worth diagramming, skip the per-system module diagram. If no flow is security-critical, skip the security frames — and say so in chat instead of creating empty sections. If a module has no findings, skip its frame. Every artifact must earn its place.

### Linking conventions

When a remote URL is known (the repo has a configured `origin` and a head SHA can be resolved), render `file:line-line` references as clickable links:

- GitHub: `https://{host}/{owner}/{repo}/blob/{sha}/{path}#L{a}-L{b}`
- GitLab: `https://{host}/{group}/{project}/-/blob/{sha}/{path}#L{a}-{b}`

Resolve the SHA with `git rev-parse HEAD`. If no remote is configured, render references as plain paths and announce once: `"No remote URL available — file references shown as plain paths."` Do not invent URLs.

### Layout reference

```
┌──────────────┐  ┌──────────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Frame 1     │  │  Frame 2     │  │ Security │  │ Security │  │ Module   │ ...
│  Overview    │→ │  Structure & │→ │ flow     │→ │ flow     │→ │ quality  │
│  (large)     │  │  Flows       │  │  frames  │  │  frames  │  │  frames  │
│              │  │  (large)     │  │          │  │          │  │          │
└──────────────┘  └──────────────┘  └──────────┘  └──────────┘  └──────────┘
```
