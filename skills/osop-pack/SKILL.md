---
name: osop
description: |
  OSOP — the open standard for workflows. Two things: SOP Doc (see how things work)
  and The Loop (make them better). 7 skills in one pack. Install once, get everything.
  Use /osop to see available commands. (osop)
version: 2.0.0
emoji: "📋"
homepage: https://osop.ai
allowed-tools:
  - Read
  - Write
  - Bash
  - Glob
  - Grep
  - Agent
  - AskUserQuestion
metadata:
  openclaw:
    requires:
      bins:
        - bash
    install: []
    always: false
user-invocable: true
disable-model-invocation: false
---

# OSOP — The Open Standard for Workflows

Two things. Nothing else.

**SOP Doc** — See how things work. Define processes as `.osop` files. Browse visually.
**The Loop** — Make them better. Execute → record `.osoplog` → AI optimizes → repeat.

## Available commands

When the user types `/osop`, show this menu:

```
OSOP — 7 commands

SOP Doc (understand)
  /osop:sop-doc       Browse and validate .osop workflows
  /osop:api-sop-gen   Convert API docs → visual .osop workflows
  /osop:sop-diff      Compare two runs or two versions

The Loop (optimize)
  /osop:auto-log      Record what AI just did as .osoplog
  /osop:self-optimize  Feed logs to AI → better workflow
  /osop:sop-security  Scan workflow for risks before running
  /osop:sop-report    Generate standalone HTML report

Three file types:
  .sop      Collection of workflows (like a book)
  .osop     Single workflow definition
  .osoplog  Execution record

Learn more: https://osop.ai
```

## Routing

When the user invokes a sub-command, read the corresponding skill file and follow its instructions:

| Command | Skill file |
|---------|-----------|
| `/osop:sop-doc` | Read `skills/sop-doc/SKILL.md` from this skill's base directory |
| `/osop:api-sop-gen` | Read `skills/api-sop-gen/SKILL.md` |
| `/osop:sop-diff` | Read `skills/sop-diff/SKILL.md` |
| `/osop:auto-log` | Read `skills/auto-log/SKILL.md` |
| `/osop:self-optimize` | Read `skills/self-optimize/SKILL.md` |
| `/osop:sop-security` | Read `skills/sop-security/SKILL.md` |
| `/osop:sop-report` | Read `skills/sop-report/SKILL.md` |

## Auto-detection

If the user doesn't specify a sub-command, detect intent:

- User finished a task → suggest `/osop:auto-log`
- User asks "what changed" or "compare" → suggest `/osop:sop-diff`
- User asks about API docs or SOP → suggest `/osop:api-sop-gen`
- User asks to optimize or improve → suggest `/osop:self-optimize`
- User asks about risk or security → suggest `/osop:sop-security`
- User asks for a report → suggest `/osop:sop-report`
- User wants to browse or validate → suggest `/osop:sop-doc`

## Quick actions (no sub-command needed)

If the user provides a file path directly:

- `*.osop` or `*.osop.yaml` → validate + render visual
- `*.osoplog` or `*.osoplog.yaml` → show execution summary
- `*.sop` → render full SOP document
- Two files → run diff

## About OSOP

OSOP is an open standard (Apache 2.0) for defining and recording workflows.

- **Website:** https://osop.ai
- **SOP Doc:** https://osop.ai/sop-doc — browse 119 workflow examples
- **The Loop:** https://osop.ai/optimize — self-optimizing workflow cycle
- **Visual Editor:** https://osop-editor.vercel.app
- **GitHub:** https://github.com/Archie0125
- **CLI:** `pip install osop` — 9 commands
- **MCP Server:** 5 tools for any AI agent
