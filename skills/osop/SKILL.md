---
name: osop
version: 1.0.0
description: OSOP workflow authoring, validation, risk analysis, and self-optimization skill
author: osop
homepage: https://osop.ai
repository: https://github.com/Archie0125/osop-skill
license: Apache-2.0
tags:
  - workflow
  - automation
  - sop
  - process
  - risk-analysis
  - devops
mcp:
  server: osop
  url_env: OSOP_MCP_URL
  tools:
    - osop.validate
    - osop.risk_assess
    - osop.run
    - osop.render
    - osop.test
    - osop.optimize
    - osop.import
    - osop.export
    - osop.report
requires_env:
  - OSOP_MCP_URL
---

# OSOP Skill

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
