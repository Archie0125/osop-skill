# OSOP Skill for Claude Code

## What This Does

This skill teaches Claude Code to produce **OSOP session logs** after significant work — structured `.osop` + `.osoplog.yaml` files that record what was done, step by step. These files can be visualized at https://osop-editor.vercel.app with risk analysis, step-by-step replay, and sub-agent tracking.

## How to Use in Any Project

Copy this file (or the content below) into your project's `CLAUDE.md`:h

---

## Session Logging with OSOP

After completing a significant task (multi-step work, not trivial one-liners), produce an OSOP session log:

1. Create a `.osop` file describing the workflow you followed
2. Create a `.osoplog.yaml` file recording what you actually did
3. Tell the user to open both files at https://osop-editor.vercel.app

### .osop format

```yaml
osop_version: "1.0"
id: "session-<short-description>"
name: "<What you did>"
description: "<1-2 sentence summary>"
version: "1.0.0"
tags: [claude-code, <relevant-tags>]

nodes:
  - id: "<step-id>"
    type: "<node-type>"
    name: "<Step Name>"
    description: "<What this step does>"
    security:
      risk_level: "<low|medium|high|critical>"

edges:
  - from: "<step-a>"
    to: "<step-b>"
    mode: "sequential"
```

### .osoplog.yaml format

```yaml
osoplog_version: "1.0"
run_id: "<uuid>"
workflow_id: "<matches .osop id>"
mode: "live"
status: "COMPLETED"
trigger:
  type: "manual"
  actor: "user"
  timestamp: "<ISO timestamp>"
started_at: "<ISO timestamp>"
ended_at: "<ISO timestamp>"
duration_ms: <total ms>
runtime:
  agent: "claude-code"
  model: "<model>"

node_records:
  - node_id: "<step-id>"
    node_type: "<type>"
    attempt: 1
    status: "COMPLETED"
    started_at: "<ISO>"
    ended_at: "<ISO>"
    duration_ms: <ms>
    outputs:
      <what you produced>
    tools_used:
      - { tool: "<tool-name>", calls: <n> }
    reasoning:
      question: "<what decision was made>"
      selected: "<chosen approach>"
      confidence: <0-1>

result_summary: "<1-2 sentence summary>"
```

### Node type mapping

| Claude Code Action | OSOP Node Type | Subtype |
|---|---|---|
| Read/explore files | `mcp` | `tool` |
| Edit/write files | `mcp` | `tool` |
| Run shell commands | `cli` | `script` |
| Run tests | `cicd` | `test` |
| Git operations | `git` | `commit`/`branch`/`pr` |
| Analyze/reason | `agent` | `llm` |
| Search codebase | `mcp` | `tool` |
| Ask user | `human` | `input` |
| User reviews | `human` | `review` |
| Spawn sub-agent | `agent` | `explore`/`plan`/`worker` |
| API calls | `api` | `rest` |

### Sub-agent tracking

When you spawn sub-agents, use `parent` field on child nodes and `spawn` edge mode:

```yaml
nodes:
  - id: "coordinator"
    type: "agent"
    subtype: "coordinator"
  - id: "explore_1"
    type: "agent"
    subtype: "explore"
    parent: "coordinator"

edges:
  - from: "coordinator"
    to: "explore_1"
    mode: "spawn"
```

In .osoplog, add parent_id and spawn_index:
```yaml
node_records:
  - node_id: "explore_1"
    parent_id: "coordinator"
    spawn_index: 1
```

### Where to save

Save to `sessions/` in the project root:
- `sessions/YYYY-MM-DD-<short-desc>.osop`
- `sessions/YYYY-MM-DD-<short-desc>.osoplog.yaml`

### When to generate

- After multi-step tasks (3+ distinct steps)
- After debugging sessions
- After feature implementations
- After refactoring work
- When the user asks "what did you do?"

### Converting to HTML report

After creating the .osop and .osoplog.yaml files, generate an HTML report:

```bash
python osop-report.py <file.osop> [file.osoplog.yaml] -o <output.html>
```

If the `osop-report.py` script is not available locally, the user can view the files at https://osop-editor.vercel.app (drag and drop both files).

The HTML report is self-contained (<15KB), supports dark mode, and shows expandable node details with tool usage, AI reasoning, and execution status.
