# OSOP — Structured Session Logs for AI Coding Agents

Your AI coding assistant does complex multi-step work. But what did it actually do? OSOP gives you **structured, portable execution logs** — not chat transcripts.

## Install (One Line)

**Option A — Copy CLAUDE.md into any project:**
```bash
curl -sL https://raw.githubusercontent.com/Archie0125/osop-openclaw-skill/main/CLAUDE.md >> CLAUDE.md
```

**Option B — Claude Code plugin:**
```bash
claude /install-plugin https://github.com/Archie0125/osop-openclaw-skill
```

**Option C — OpenClaw:**
```bash
clawhub install osop
```

That's it. Claude Code will now generate `.osop` + `.osoplog.yaml` after multi-step tasks.

## What You Get

```
Before: 200-line chat transcript, hard to skim
After:   Structured workflow + execution record → visual HTML report
```

### Two Files

| File | What | Example |
|------|------|---------|
| `.osop.yaml` | What was **planned** — DAG of steps | "Explore → Implement → Test → Review" |
| `.osoplog.yaml` | What **actually happened** — timestamps, tools, outputs | "Read 4 files, edited 2, ran tests in 3.2s" |

### View Reports

Drag both files into https://osop-editor.vercel.app for:
- Visual DAG diagram
- Step-by-step execution timeline
- Tool usage breakdown
- Risk analysis overlay

## OSOP Core — 4 Types, 4 Modes

OSOP uses a minimal schema for AI agent workflows:

**Node types:** `agent` (AI/LLM), `api` (HTTP), `cli` (shell), `human` (manual)
**Edge modes:** `sequential`, `conditional`, `parallel`, `fallback`

A complete workflow fits in 15-25 lines of YAML.

## Skills

| Skill | What it does |
|-------|-------------|
| `/osop` | Validate + render any .osop file |
| `/osop:auto-log` | Auto-generate .osoplog after task completion |
| `/osop:sop-report` | Generate standalone HTML report |
| `/osop:sop-security` | Security risk scan before execution |
| `/osop:sop-diff` | Compare two executions |

## Works With

Claude Code, Cursor, Windsurf, Codex, Copilot, Cline, Aider, Continue.dev, Devin, Zed, and more.

## Links

- **Visual Editor:** [osop-editor.vercel.app](https://osop-editor.vercel.app)
- **Spec:** [github.com/Archie0125/osop-spec](https://github.com/Archie0125/osop-spec)
- **Examples:** [github.com/Archie0125/osop-examples](https://github.com/Archie0125/osop-examples)

## License

Apache 2.0
