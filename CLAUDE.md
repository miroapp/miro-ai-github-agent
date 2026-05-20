# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

This is a **GitHub AgentHQ Plugin** that ships Miro-specific agentic capabilities (custom agents, skills, hooks, MCP server configs) into the Copilot harness. The plugin is wrapped in an **Agentic App** (a GitHub App, `type=plugin`) so it has a first-class identity on GitHub — it can be @-mentioned, assigned to issues, surfaced in Mission Control, and respond to platform triggers.

The agentic loop, model orchestration, and infrastructure are provided by the GitHub Copilot harness. This repo only provides the *what*: domain knowledge, tooling (MCP), workflows.

Authoritative spec: [Copilot CLI plugin documentation](https://docs.github.com/en/copilot/concepts/agents/about-copilot-coding-agent). Treat the plugin spec as the source of truth — if a convention here conflicts with the spec, the spec wins.

## Repository layout

```
plugin.json          # Required manifest (declares name, version, paths)
agents/              # Custom agents — each is a *.agent.md file with YAML frontmatter
  main.agent.md      # The main agent (entry point invoked by @-mention, assignment, etc.)
skills/              # Skills — each in skills/<name>/SKILL.md
hooks.json           # Deterministic event hooks (e.g., before:tool:bash)
```

One Agentic App points to **one** public repo containing **one** plugin. This repo must remain public for platform sync.

## Architecture notes

- **Main agent** (`agents/main.agent.md`) is the entry point. The `name:` in its frontmatter is what gets registered. Additional `*.agent.md` files are sub-agents the main agent can delegate to via inference — they are *not* directly addressable from outside the plugin.
- **MCP servers** are declared in agent frontmatter under `mcp-servers:`, not in `plugin.json`. The current main agent wires up the **Miro MCP server** at `https://mcp.miro.com/` with `oidc: true`.
- **GitHub MCP server** is pre-installed and authenticated by the harness — do *not* declare it in agent frontmatter. It's scoped to the current job's repo + user.
- **OIDC for MCP tools**: when `oidc: true` is set, the harness signs a short-lived JWT (5 min, RFC 7523 jwt-bearer assertion) and exchanges it at the MCP server's token endpoint for an access token. No long-lived secrets. Auto-discovery uses `/.well-known/oauth-authorization-server`, `/.well-known/oauth-protected-resource`, then `/.well-known/openid-configuration`. Override via `oidc.endpoints.{base,exchange,revoke,failure}`. Use `grant-type: client_credentials` for Entra-style IdPs.
- **Hooks** in `hooks.json` run deterministically on platform events (`before:tool:bash`, etc.) — use these for validation/policy enforcement, not for agent logic.
- **Progress reporting & Mission Control** are handled automatically by the harness. Do not add custom progress-reporting code.

## Editing conventions

- **Agent frontmatter** uses YAML; the body below `---` is the system prompt. Keep the prompt focused on *what this agent does* — capabilities, decision rules, when to delegate. Do not duplicate harness behavior (model selection, progress reporting).
- **Skill frontmatter** must include `name`, `description`, and typically `disable-model-invocation: true`. The body is instructions for the skill.
- When adding a new agent, give it a clear `description:` — the main agent uses descriptions to decide which sub-agent to invoke via inference.
- When adding a new MCP server, prefer `oidc: true` over static tokens. Pin `tools: ["*"]` only when you genuinely need every tool; otherwise list the specific tool names to narrow blast radius.

## Local development & testing

The Copilot CLI installs the plugin from a local path without needing to deploy the Agentic App:

```bash
# Install from this directory
copilot plugin install .

# Verify it loaded
copilot plugin list

# In an interactive session:
/agent           # verify the main agent (and any sub-agents) loaded
/skills list     # verify skills loaded

# Re-install after editing (components are cached)
copilot plugin install .

# Uninstall
copilot plugin uninstall miro-ai-github-agent
```

You can also install directly from GitHub once pushed: `copilot plugin install miroapp/miro-ai-github-agent`.

## Deploying

1. Push to `miroapp/miro-ai-github-agent` (repo must be **public**).
2. Register a GitHub App with the permissions the agent needs.
3. Configure it as an Agentic App (`type=plugin`) pointing at this repo.
4. The platform syncs `plugin.json` and the app becomes available via its GitHub identity.

## Common pitfalls

- Forgetting to bump `version` in `plugin.json` after meaningful changes — the platform uses it for sync.
- Declaring the GitHub MCP server in an agent (it's pre-installed; declaring it shadows the harness-provided one).
- Adding long-lived secrets to MCP server configs — use OIDC token exchange instead.
- Making the repo private — distribution will silently break.
