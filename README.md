# miro-ai-github-agent

A **GitHub AgentHQ Plugin** that brings Miro-specific agentic capabilities into GitHub Copilot. The plugin runs inside the Copilot harness and is exposed on GitHub as an **Agentic App** — so it can be @-mentioned, assigned to issues, invoked from Mission Control, and triggered by platform events.

## What's in the box

| Component | Path | Purpose |
|-----------|------|---------|
| Manifest | `plugin.json` | Declares the plugin name, version, and component paths |
| Main agent | `agents/main.agent.md` | Entry point invoked when the app's identity is addressed on GitHub |
| Skills | `skills/<name>/SKILL.md` | Discrete, callable capabilities the agents can invoke |
| Hooks | `hooks.json` | Deterministic event handlers (e.g. `before:tool:bash`) |

## MCP integrations

The main agent is wired to the **Miro MCP server** at `https://mcp.miro.com/` (HTTP transport, OIDC-authenticated). It exposes Miro's API surface — boards, items, docs, diagrams, tables, work items, comments, etc. — as tools the agent can call via inference.

The **GitHub MCP server** is pre-installed and authenticated by the Copilot harness, so the agent can read repo contents, manage issues/PRs, and post review comments without any additional configuration.

### Authentication

OIDC follows the workflow identity federation pattern (RFC 7523):

1. The harness signs a short-lived JWT (5 min) with claims about the workload — agent identity, repository, organization, acting user.
2. It POSTs the JWT to the MCP server's token-exchange endpoint (auto-discovered via `/.well-known/oauth-authorization-server`, or override with `oidc.endpoints.exchange`).
3. The MCP server verifies the JWT against GitHub's public keys, authorizes the request, and returns an access token.
4. The harness injects the access token into the MCP server (as a `Bearer` header for remote servers).
5. On job completion, the harness optionally calls the revocation endpoint.

No long-lived secrets are stored. See `CLAUDE.md` and the [Integration Guide](https://docs.github.com/en/copilot/concepts/agents/about-copilot-coding-agent) for the full list of OIDC config options.

## Local development

The [Copilot CLI](https://docs.github.com/en/copilot/github-copilot-in-the-cli) lets you install and test the plugin without deploying the Agentic App.

```bash
# Install from this directory
copilot plugin install .

# Verify it loaded
copilot plugin list

# In an interactive session
/agent              # verify agents loaded
/skills list        # verify skills loaded

# Re-install after edits (components are cached)
copilot plugin install .

# Uninstall when done
copilot plugin uninstall miro-ai-github-agent
```

You can also install straight from the GitHub repo:

```bash
copilot plugin install miroapp/miro-ai-github-agent
```

## Deploying as an Agentic App

1. Push this repo to `miroapp/miro-ai-github-agent`. **The repo must be public** for the platform to sync and distribute it across environments (Cloud and Proxima tenants).
2. Register a GitHub App with the permissions the agent needs.
3. Configure it as an **Agentic App** (`type=plugin`) pointing at this public repository.
4. The platform syncs `plugin.json` and the app becomes available on GitHub via its identity (@-mention, assignment, Mission Control, triggers).

One Agentic App ↔ one public repo ↔ one plugin. This is not a marketplace of plugins.

## Adding a new agent

Drop a new `*.agent.md` file in `agents/`:

```markdown
---
name: my-subagent
description: Short description used by the main agent to decide when to delegate
disable-model-invocation: true
tools: ["bash", "view", "edit"]
---

System prompt for this agent…
```

The main agent will pick it up via inference when its `description:` matches a task.

## Adding a new skill

Create `skills/<skill-name>/SKILL.md`:

```markdown
---
name: my-skill
description: What this skill does
disable-model-invocation: true
---

Instructions for the skill…
```

## Adding a hook

Edit `hooks.json`:

```json
{
  "hooks": [
    { "event": "before:tool:bash", "command": "validate-command.sh" }
  ]
}
```

## References

- [GitHub Copilot CLI plugin spec](https://docs.github.com/en/copilot/github-copilot-in-the-cli/configuring-github-copilot-in-the-cli)
- [Miro MCP](https://mcp.miro.com/)
- RFC 7523 (JWT bearer assertions), RFC 8693 (token exchange), RFC 7009 (token revocation)
