# OSOP Skills for Claude Code

[![OSOP Compatible](https://img.shields.io/badge/OSOP-compatible-blue)](https://osop.ai)

Three Claude Code skills for workflow logging, risk review, and self-optimization.

## Install

```bash
claude /install-plugin https://github.com/Archie0125/osop-skill
```

Or manually: clone this repo and run Claude Code with `--plugin-dir`:
```bash
git clone https://github.com/Archie0125/osop-skill.git
claude --plugin-dir ./osop-skill
```

## Skills

### `/osop:osop-log` — Session Logging

After completing a task, run:
```
/osop:osop-log fixed auth token refresh race condition
```

Creates structured `.osop` + `.osoplog.yaml` in `sessions/` recording every step, tool used, and decision made. View at https://osop-editor.vercel.app with step-by-step replay.

### `/osop:osop-review` — Risk Review

Before running someone else's workflow:
```
/osop:osop-review path/to/workflow.osop
```

Analyzes for: destructive commands, overly broad permissions, missing approval gates, hardcoded secrets, unbounded costs. Returns a risk score (0-100) with verdict.

### `/osop:osop-optimize` — Self-Optimization

After running a workflow multiple times:
```
/osop:osop-optimize path/to/workflow.osop
```

Reads past `.osoplog` files, identifies slow steps and failure hotspots, suggests improvements (retry policies, parallelization, timeouts), and can apply changes with your approval.

## Alternative: Drop-in CLAUDE.md

If you just want session logging without installing the plugin, copy `CLAUDE.md` into your project:

```bash
curl -O https://raw.githubusercontent.com/Archie0125/osop-skill/main/CLAUDE.md
```

## Structure

```
.claude-plugin/plugin.json         # Claude Code plugin manifest
skills/
  osop-log/SKILL.md                # /osop:osop-log — session logging
  osop-review/SKILL.md             # /osop:osop-review — risk review
  osop-optimize/SKILL.md           # /osop:osop-optimize — self-optimization
prompts/
  system-prompt.md                 # Full OSOP knowledge base (16 node types, 13 edge modes)
  policy-pack.md                   # 9 safety policies
examples/
  fix-auth-bug.osop                # Example workflow with sub-agents
  fix-auth-bug.osoplog.yaml        # Example execution log
CLAUDE.md                          # Drop-in alternative (no plugin needed)
```

## Visualize Logs

Open `.osop` + `.osoplog.yaml` at **https://osop-editor.vercel.app**:
- Step-by-step replay with play/pause
- Risk analysis overlay
- Sub-agent hierarchy tree
- Tool usage per step
- AI reasoning decisions

## License

Apache License 2.0
