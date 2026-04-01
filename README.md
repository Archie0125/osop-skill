# OSOP Skill

[![OSOP Compatible](https://img.shields.io/badge/OSOP-compatible-blue)](https://osop.ai)

AI coding assistant skill for structured workflow logging and reporting. Works with **Claude Code, Codex, Grok, Cursor, OpenClaw** — any tool that reads markdown instructions or supports Claude Code plugins.

## What It Does

1. **Creates `.osop` + `.osoplog.yaml`** — structured records of what the AI did, step by step
2. **Converts to HTML reports** — standalone, dark-mode, expandable, opens in any browser

## Install

### Claude Code (Plugin)
```bash
claude /install-plugin https://github.com/Archie0125/osop-skill
```

Gives you 4 slash commands:
- `/osop:osop-log` — Record what you just did
- `/osop:osop-report` — Convert .osop to HTML
- `/osop:osop-review` — Risk-analyze a workflow
- `/osop:osop-optimize` — Improve workflow from history

### Any AI Tool (Drop-in)

Copy the content of `CLAUDE.md` into your AI tool's system prompt or project instructions file. Works with Codex, Grok, Cursor, or any tool that reads markdown instructions.

```bash
curl -O https://raw.githubusercontent.com/Archie0125/osop-skill/main/CLAUDE.md
```

### Standalone Report Generator

Generate HTML reports without any AI tool:

```bash
pip install pyyaml
python scripts/osop-report.py workflow.osop execution.osoplog.yaml -o report.html
```

## Output Examples

After a session, you get:

**`sessions/2026-04-01-fix-auth-bug.osop`** — workflow definition
```yaml
osop_version: "1.0"
id: session-fix-auth-bug
name: "Fix Authentication Bug"
nodes:
  - id: explore
    type: agent
    subtype: explore
    name: "Search Auth Code"
  - id: fix
    type: mcp
    name: "Write Fix"
edges:
  - from: explore
    to: fix
```

**`sessions/2026-04-01-fix-auth-bug.osoplog.yaml`** — what actually happened
```yaml
osoplog_version: "1.0"
status: COMPLETED
duration_ms: 930000
node_records:
  - node_id: explore
    status: COMPLETED
    tools_used:
      - { tool: Grep, calls: 5 }
      - { tool: Read, calls: 4 }
    reasoning:
      question: "Where is the token refresh logic?"
      selected: "src/auth/token-refresh.ts"
```

**`report.html`** — self-contained visual report (dark mode, <15KB, zero deps)

## Structure

```
.claude-plugin/plugin.json         # Claude Code plugin manifest
skills/
  osop-log/SKILL.md                # /osop:osop-log — session logging
  osop-report/SKILL.md             # /osop:osop-report — HTML generation
  osop-review/SKILL.md             # /osop:osop-review — risk analysis
  osop-optimize/SKILL.md           # /osop:osop-optimize — self-improvement
scripts/
  osop-report.py                   # Standalone HTML report generator (Python)
prompts/
  system-prompt.md                 # Full OSOP knowledge (16 node types, 13 edge modes)
  policy-pack.md                   # 9 safety policies
examples/
  fix-auth-bug.osop                # Example with sub-agents + spawn edges
  fix-auth-bug.osoplog.yaml        # Full execution log with tools, reasoning
CLAUDE.md                          # Drop-in for any AI tool
```

## View Reports

- Open HTML files in any browser
- Or upload `.osop` + `.osoplog.yaml` to **https://osop-editor.vercel.app** for interactive replay

## License

Apache License 2.0
