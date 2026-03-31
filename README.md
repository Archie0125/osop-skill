# OSOP Skill for OpenClaw

[![License](https://img.shields.io/badge/License-Apache_2.0-blue.svg)](LICENSE)

OSOP skill pack for [OpenClaw](https://openclaw.ai) — provides a system prompt, policy pack, and configuration for AI agents to author, validate, and execute OSOP workflows via MCP integration.

Website: [osop.ai](https://osop.ai) | GitHub: [github.com/osop/osop-openclaw-skill](https://github.com/osop/osop-openclaw-skill)

## What This Skill Does

When installed, this skill teaches an OpenClaw-compatible AI agent to:

1. **Understand OSOP** — the schema, node types, edge modes, and contracts
2. **Author workflows** — generate valid `.osop.yaml` files from natural language descriptions
3. **Validate and test** — use MCP tools to validate workflows against the schema
4. **Follow safety policies** — enforce approval gates for high-risk operations, require dry-runs before production execution

## Structure

```
skills/osop/
  SKILL.md          # OpenClaw skill definition with frontmatter
  metadata.json     # Skill metadata and environment gating
prompts/
  system-prompt.md  # Core system prompt for OSOP-aware agents
  policy-pack.md    # Safety policies for high-risk workflow operations
```

## Installation

### OpenClaw CLI

```bash
openclaw skill install osop/osop-openclaw-skill
```

### Manual

Copy the `skills/osop/` directory into your OpenClaw skills folder and set the `OSOP_MCP_URL` environment variable to point to your running OSOP MCP server.

## Environment Variables

| Variable | Required | Description |
|----------|----------|-------------|
| `OSOP_MCP_URL` | Yes | URL of the OSOP MCP server (e.g., `http://localhost:8080`) |
| `OSOP_APPROVAL_MODE` | No | `auto` or `manual` — controls whether approval nodes require human input. Default: `manual` |

## License

Apache License 2.0 — see [LICENSE](LICENSE) for details.
