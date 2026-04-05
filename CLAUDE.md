# OSOP Session Logging for Claude Code

## What This Does

After completing significant work (3+ steps), Claude Code generates a structured session log — `.osop` + `.osoplog.yaml` files that record what was done. View at https://osop-editor.vercel.app

## Quick Start

Copy this file into any project's `CLAUDE.md` to enable session logging.

---

## Session Logging Instructions

After completing a significant task, produce two files:

### 1. Workflow Definition (`.osop`)

```yaml
osop_version: "1.0"
id: "session-<short-description>"
name: "<What you did>"
description: "<1-2 sentence summary>"
tags: [claude-code, <relevant-tags>]

nodes:
  - id: "<step-id>"
    type: "<agent|api|cli|human>"
    name: "<Step Name>"
    description: "<What this step does>"

edges:
  - from: "<step-a>"
    to: "<step-b>"
```

### 2. Execution Record (`.osoplog.yaml`)

```yaml
osoplog_version: "1.0"
run_id: "<uuid>"
workflow_id: "<matches .osop id>"
status: "COMPLETED"
started_at: "<ISO timestamp>"
ended_at: "<ISO timestamp>"
duration_ms: <total ms>

runtime:
  agent: "claude-code"
  model: "<model you are running as>"

node_records:
  - node_id: "<step-id>"
    status: "COMPLETED"
    started_at: "<ISO>"
    ended_at: "<ISO>"
    duration_ms: <ms>
    outputs:
      <what you produced>
    tools_used:
      - { tool: "<tool-name>", calls: <n> }

result_summary: "<1-2 sentence summary>"
```

### Node Type Mapping (OSOP Core)

| Claude Code Action | Type | Subtype |
|---|---|---|
| Read/explore/search files | `agent` | `llm` |
| Edit/write files | `agent` | `llm` |
| Run shell commands | `cli` | `script` |
| Run tests | `cli` | `test` |
| Analyze/reason about code | `agent` | `llm` |
| Ask user a question | `human` | `input` |
| User reviews/approves | `human` | `review` |
| API calls (web fetch) | `api` | `rest` |
| Spawn sub-agent | `agent` | `explore` or `plan` |

### Edge Modes

| Pattern | Mode |
|---|---|
| Step A then Step B | `sequential` (default) |
| A and B at same time | `parallel` |
| If condition... | `conditional` + `when:` |
| On failure... | `fallback` |

### Where to Save

Save to `sessions/` in the project root:
- `sessions/YYYY-MM-DD-<short-desc>.osop.yaml`
- `sessions/YYYY-MM-DD-<short-desc>.osoplog.yaml`

### When to Generate

- After multi-step tasks (3+ distinct steps)
- After debugging sessions
- After feature implementations
- When the user asks "what did you do?"

### Viewing Reports

Drag and drop both files at https://osop-editor.vercel.app for visual reports with risk analysis, step-by-step replay, and execution timeline.
