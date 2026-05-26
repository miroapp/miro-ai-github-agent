---
name: miro-way-of-the-code-implement
description: Use after the user has modified the "Future Architecture" frame on a Miro board (created by miro-way-of-the-code-extract). Reads the diffs between the Current and Future frames, translates the deltas into code changes where possible, and opens a PR whose description maps each diagram delta to its corresponding code change or to a clear TODO for human follow-up.
---

# Way of the Code — Implement

Read the **Current Architecture** and **Future Architecture** frames from a Miro board, diff the paired Mermaid diagrams, translate the deltas into code changes (when concrete enough), and open a PR whose description maps every delta to either a code change or a clear TODO. The PR description is the primary artifact — when deltas are ambiguous, the PR carries a structured implementation plan rather than guessed code.

The user provides the Miro board URL. The user should already have edited the Future Architecture frame; if they haven't, this skill will detect the empty delta and report it.

## Workflow

### 1. Locate the two frames

Use `board_list_items` (or `context_get` for a fuller view) to find the two frames on the supplied board:

- A frame with title **`Current Architecture`**
- A frame with title **`Future Architecture`**

If either is missing, stop and report in chat:

> Could not find both 'Current Architecture' and 'Future Architecture' frames on the board — was it set up with miro-way-of-the-code-extract?

Capture each frame's ID as `CURRENT_FRAME_ID` and `FUTURE_FRAME_ID`.

### 2. List the items in each frame

For each frame, list its child items and pair section headers with the diagram that immediately follows. Build two parallel maps keyed by header text:

```
current_by_title: { "<header text>": <diagram_id> }
future_by_title:  { "<header text>": <diagram_id> }
```

**Tool reference for this step:**

| Tool | Used for |
|---|---|
| `board_list_items` | Enumerate child items in each frame |
| `context_get` / `context_explore` | Fuller context if `board_list_items` is ambiguous |
| `doc_get` | Read each section header's text |
| `diagram_get_dsl` | Fetch the Mermaid DSL for each paired diagram (used in §3) |

### 3. Compute the delta per section

For every title key across the two maps:

| Case | Detection | Treatment |
|---|---|---|
| **Modified** | Title in both maps | Fetch both diagrams' Mermaid DSL via `diagram_get_dsl`, then structural-diff |
| **Added** | Title only in `future_by_title` | The user is introducing a new system/module/flow |
| **Removed** | Title only in `current_by_title` | The user is removing a system/module/flow |

For modified diagrams, prefer a **structural** diff over a text diff — parse the Mermaid DSL into nodes and edges and report:

- **Nodes added** — present in future, absent in current
- **Nodes removed**
- **Nodes renamed** — heuristic: edge structure preserved, single label difference
- **Edges added** / **removed** / **redirected**
- **Subgraph / cluster changes**

If the DSL is too complex to parse cleanly (rare, but possible for deeply nested subgraphs or custom Mermaid extensions), fall back to a text diff and note once in chat: `"Falling back to text diff for <section name> — structural parse failed."`

If `current_by_title` and `future_by_title` are byte-identical (no edits at all), stop and report:

> The Future Architecture frame is unchanged from Current. Nothing to implement — edit the Future frame and re-run.

### 4. Translate deltas to code changes

For each delta, decide whether the code change is concrete enough to make. Use these guidelines as a starting point — apply judgment:

| Delta | Likely code change |
|---|---|
| New system node | New top-level directory (e.g. `services/<new>` or `apps/<new>`), scaffolded with the minimal manifest for the system's type |
| Removed system node | Delete the corresponding directory plus any imports/calls from sibling systems |
| New module node | New file or package directory inside the parent system |
| Renamed module | File/directory rename + import-path updates across the system |
| New edge between systems | Wire up a client per the edge's surface label (HTTP fetch, gRPC stub, workspace import) |
| Removed edge | Remove the client wiring and any newly-dead code paths |
| New flow step (sequence diagram) | New handler / function / method in the actor's system |
| Reordered flow | Reorder calls in the orchestrating handler |
| New subgraph | New package/module boundary; reshuffle existing files into it if the future diagram shows that |

**When a delta is ambiguous** — a new node with no obvious file mapping, a renamed component you can't safely match, a flow step that requires non-trivial design decisions — do NOT guess. Leave the working tree untouched for that delta and record it as a TODO for the PR description (§6).

### 5. Make the code changes

For each delta where the change is concrete:

1. Create / modify / delete the relevant files
2. Update imports, route tables, service registries, manifests
3. Don't touch tests unless an existing test obviously breaks; if a test breaks, note it as a TODO rather than deleting or rewriting it

For ambiguous deltas, do nothing in the working tree — they go in the PR description as planned work.

### 6. Open the PR

#### 6a. Branch and commit

Create a branch named:

```
wotc/architecture-<short-hash>
```

where `<short-hash>` is the first 7 characters of `FUTURE_FRAME_ID`. This makes re-runs deterministic — running the skill twice on the same Future frame updates the same branch.

If the branch already exists locally or remotely, check it out and update it in place (force-push later — see §6c). Don't create a parallel branch.

Commit the working-tree changes with a message like:

```
WotC: implement architectural changes from <BOARD_URL>

<one-line summary of the dominant change>
```

If there were no working-tree changes (everything was TODO), make an empty commit (`git commit --allow-empty`) — the PR still needs to exist as a planning artifact.

#### 6b. Push and open the PR

Push the branch. Open the PR via the **GitHub MCP server** (pre-installed by the Copilot harness, so no auth setup needed). If running from the Copilot CLI locally without the GitHub MCP wired up, fall back to `gh pr create`.

The PR description follows this template:

```markdown
## Architectural change from Miro

**Board:** <BOARD_URL>
**Future frame:** <BOARD_URL>?moveToWidget=<FUTURE_FRAME_ID>

## Summary

<2–4 sentences describing the overall shape of the change, derived from the deltas in §3>

## Deltas detected

### <Section header 1>

- **Added:** <bullet list of added nodes/edges, or "—" if none>
- **Removed:** <bullet list, or "—">
- **Renamed:** <bullet list, or "—">

**Code changes:** <bullet list of files modified/created/deleted, with file:line references, or "none — see Outstanding below">

**Outstanding:** <bullet list of deltas in this section that need human follow-up; be concrete about what's unclear. Omit if none.>

### <Section header 2>

…

## Verification checklist

- [ ] Build passes
- [ ] Tests pass / new tests added where appropriate
- [ ] Outstanding items above are resolved or explicitly deferred
- [ ] Diagram pairing on the board still matches what was implemented
```

If there were **no implementable deltas** (everything ended up under Outstanding), add this near the top of the description, right after the Summary:

> ⚠️ No code changes were made — every delta required design decisions a human should drive. This PR is a planning artifact; treat it as a spec.

…and label the PR `architecture-plan` so it's clear the diff is intentionally empty.

#### 6c. Re-runnable

This skill is safe to re-run. If the branch already exists (the user edited the Future frame again and wants the PR refreshed), update it in place and **force-push with `--force-with-lease`**. Rewrite the PR description from scratch — don't try to merge with the previous one, since the deltas may have shifted entirely.

### 7. Output

Report in chat:

1. PR URL
2. Branch name
3. Summary: `<N> sections analyzed, <M> deltas detected, <K> code changes made, <O> outstanding items`
4. If any deltas were marked outstanding, list each with a one-line reason so the user can decide whether to pick them up manually or refine the Future frame and re-run the skill

## Background

### The PR description is the contract

The most important output of this skill is the PR description. Code changes are best-effort; the description is the spec. A well-written description means a human reviewer (or the user themselves) can finish what the skill couldn't implement automatically. Don't skimp on the per-section breakdowns.

### Trust boundaries

The skill operates on whatever the user put in the Future frame. It does **not** validate that the proposed architecture is good — it translates intent into code as faithfully as it can and leaves judgment to PR review. If a delta would make a system insecure or unrunnable, the PR description should flag it but the change still goes in (and is rejected at PR review, or fixed there).

### When the board is in a bad state

If the two frames have wildly different titles, the Future frame has no headers at all, or pairs of diagrams can't be matched, **stop after §3** and report the structural issues in chat. Don't open a PR for a board that wasn't edited per the extract skill's convention — the user should fix the board (or re-run the extract skill) before re-running this one.

### What this skill is not

This skill is not a substitute for design review. It mechanically translates diagram deltas into code; it does not check whether the resulting architecture is correct, performant, or wise. Those are the reviewer's job. The skill's value is in making the architecture → code translation cheap enough to iterate on.
