---
name: osop
version: 0.1.0
description: OSOP workflow authoring, validation, and execution skill
author: osop
homepage: https://osop.ai
repository: https://github.com/osop/osop-openclaw-skill
license: Apache-2.0
tags:
  - workflow
  - automation
  - sop
  - process
  - devops
mcp:
  server: osop
  url_env: OSOP_MCP_URL
  tools:
    - osop.validate
    - osop.run
    - osop.render
    - osop.test
    - osop.optimize
    - osop.import
    - osop.export
requires_env:
  - OSOP_MCP_URL
---

# OSOP Skill

This skill enables AI agents to work with OSOP (Open Standard Operating Procedures) — a universal protocol for process definitions.

## Capabilities

When this skill is active, the agent can:

- **Create** OSOP workflow definitions from natural language descriptions
- **Validate** workflow YAML against the OSOP schema using `osop.validate`
- **Execute** workflows with `osop.run`, including dry-run mode for safety
- **Render** workflows as Mermaid diagrams using `osop.render`
- **Test** workflows against defined test cases with `osop.test`
- **Optimize** workflows for parallelism and efficiency with `osop.optimize`
- **Convert** between OSOP and external formats (GitHub Actions, BPMN, Airflow) using `osop.import` and `osop.export`

## Behavior Guidelines

1. Always validate workflows before running them
2. Use dry-run mode before executing workflows with side effects
3. Require explicit user approval before running workflows that modify production systems
4. Follow the policy pack for high-risk operations
5. Render diagrams when explaining workflow structure to users

## OSOP Node Types

The 12 supported node types are: `start`, `end`, `step`, `decision`, `fork`, `join`, `loop`, `retry`, `approval`, `webhook`, `timer`, `subprocess`.

## Edge Modes

Edges connect nodes and support modes: `default`, `conditional`, `error`, `timeout`.

## Example Interaction

**User:** Create a CI/CD workflow that builds, tests, and deploys with manual approval.

**Agent:** Uses OSOP schema knowledge to generate a valid `.osop.yaml`, validates it via `osop.validate`, renders a diagram via `osop.render`, and presents both to the user.
