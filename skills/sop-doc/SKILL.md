---
name: sop-doc
description: "See what any process does — browse .osop workflows visually, validate, render diagrams. Part of SOP Doc."
version: 1.2.0
emoji: "\U0001F4CB"
homepage: https://osop.ai
metadata:
  openclaw:
    requires:
      env:
        - OSOP_MCP_URL
      bins:
        - python3
      anyBins:
        - python3
        - python
      config:
        - ~/.osop/config.yaml
    primaryEnv: OSOP_MCP_URL
    install:
      - kind: uv
        package: pyyaml
    always: false
user-invocable: true
disable-model-invocation: false
---

## Preamble (run first)

```bash
mkdir -p ~/.osop/analytics ~/.osop/projects
_OSOP_TEL=$(cat ~/.osop/config.yaml 2>/dev/null | grep telemetry | awk '{print $2}' || echo "unset")
_OSOP_TEL_PROMPTED=$([ -f ~/.osop/.telemetry-prompted ] && echo "yes" || echo "no")
_OSOP_VERSION="1.1.0"
_OSOP_SESSION_ID="$$-$(date +%s)"
_OSOP_TEL_START=$(date +%s)
echo "OSOP_TELEMETRY: $_OSOP_TEL"
echo "OSOP_TEL_PROMPTED: $_OSOP_TEL_PROMPTED"
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill sop-doc --event started --session "$_OSOP_SESSION_ID" 2>/dev/null || true
_SLUG=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null | tr '[:upper:]' '[:lower:]' | tr ' ' '-' || echo "unknown")
[ -f ~/.osop/projects/"$_SLUG"/learnings.jsonl ] && echo "--- Recent OSOP learnings ---" && tail -3 ~/.osop/projects/"$_SLUG"/learnings.jsonl 2>/dev/null || true
```

If `OSOP_TEL_PROMPTED` is `no`: use AskUserQuestion — "Help OSOP get better! Community mode shares anonymous usage data (which skills you use, how long they take) so we can improve the workflow format. No code, file paths, or repo names are ever sent."
Options: A) Help OSOP get better! → `osop-config set telemetry community` B) No thanks → fallback anonymous ask → `set telemetry off|anonymous`
Then: `touch ~/.osop/.telemetry-prompted`

# SOP Doc

# Part of SOP Doc — understand how things work

This skill enables AI agents to work with OSOP (Open Standard Operating Protocol) — a universal protocol for defining, validating, risk-assessing, and executing process workflows.

## Capabilities

When this skill is active, the agent can:

- **Create** OSOP workflow definitions from natural language descriptions
- **Validate** workflow YAML against the OSOP schema using `osop.validate`
- **Risk Assess** workflows for security issues, permission gaps, destructive commands, and cost exposure using `osop.risk_assess`
- **Execute** workflows with `osop.run`, including dry-run mode for safety
- **Render** workflows as Mermaid diagrams using `osop.render`
- **Test** workflows against defined test cases with `osop.test`
- **Optimize** workflows using execution history — detect slow steps, failure hotspots, and suggest improvements with `osop.optimize`
- **Convert** between OSOP and external formats (GitHub Actions, BPMN, Airflow) using `osop.import` and `osop.export`
- **Report** on workflow executions with `osop.report`

## OSOP Node Types (v1.0)

16 supported node types in 4 categories:

**Actors:** `human`, `agent`, `company`, `department`
**Technical:** `api`, `cli`, `db`, `git`, `docker`, `cicd`, `infra`, `mcp`
**Flow Control:** `system`, `event`, `gateway`, `data`

Use `subtype` for domain specialization (e.g., `type: agent, subtype: llm`).

## Edge Modes

13 modes: `sequential`, `conditional`, `parallel`, `loop`, `event`, `fallback`, `error`, `timeout`, `compensation`, `message`, `dataflow`, `signal`, `weighted`, `spawn`.

## Security Metadata

Nodes can declare `security.risk_level` (low/medium/high/critical), `security.permissions`, `security.secrets`, and `approval_gate` for risk analysis.

## Self-Optimization Loop

1. Execute workflow → produce `.osoplog`
2. Aggregate stats from past runs
3. Call `osop.optimize` with run history → get improvement suggestions
4. Apply approved changes → execute improved workflow
5. Repeat — the SOP gets better each iteration

## Behavior Guidelines

1. Always validate workflows before running them
2. Always assess risk before executing third-party workflows
3. Use dry-run mode before executing workflows with side effects
4. Require explicit user approval before running high-risk operations
5. Follow the policy pack for safety enforcement
6. Save `.osoplog` after every execution as evidence
7. Render diagrams when explaining workflow structure
