---
name: osop-optimize
description: Analyze past .osoplog execution records and suggest improvements to the .osop workflow. Identifies slow steps, failure hotspots, and parallelization opportunities.
argument-hint: <path-to-osop-file>
allowed-tools: Read, Glob, Grep, Write, Bash
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
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill osop-optimize --event started --session "$_OSOP_SESSION_ID" 2>/dev/null || true
echo "{\"skill\":\"osop-optimize\",\"ts\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"}" > ~/.osop/analytics/.pending-"$_OSOP_SESSION_ID" 2>/dev/null || true
_SLUG=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null | tr '[:upper:]' '[:lower:]' | tr ' ' '-' || echo "unknown")
[ -f ~/.osop/projects/"$_SLUG"/learnings.jsonl ] && echo "--- Recent OSOP learnings ---" && tail -3 ~/.osop/projects/"$_SLUG"/learnings.jsonl 2>/dev/null || true
```

If `OSOP_TEL_PROMPTED` is `no`: use AskUserQuestion — same prompt as osop-log preamble.

# OSOP Workflow Optimizer

Improve a workflow based on its execution history.

## Target workflow

$ARGUMENTS

## What to do

1. **Read the .osop file** specified in the argument

2. **Find execution logs** — look for matching `.osoplog.yaml` files in `sessions/` or the same directory

3. **Aggregate stats** from all matching logs:
   - Per-node: average duration, failure rate, timeout rate, common errors
   - Overall: success rate, average total duration, run count

4. **Identify issues**:
   - **Slow steps**: nodes with avg_duration > 5s
   - **Failure hotspots**: nodes with failure_rate > 10%
   - **Bottlenecks**: nodes that are both slow AND unreliable
   - **Missing retries**: external call nodes (api, cli, agent, infra, mcp) without retry_policy
   - **Missing timeouts**: external call nodes without timeout_sec
   - **Parallelization**: sequential chains of 3+ independent nodes (no data dependency between them)
   - **Missing error handling**: high-risk nodes without fallback/error edges

5. **Generate suggestions** and present as a table:
   ```
   | Type | Target Node | Description | Priority |
   |------|-------------|-------------|----------|
   | add_retry | fetch-data | 35% failure rate, add retry with backoff | HIGH |
   | parallelize | scan, test, lint | Independent steps, run in parallel | MEDIUM |
   ```

6. **If user approves**, apply changes to the .osop file:
   - Add `retry_policy` to failure-prone nodes
   - Add `timeout_sec` based on observed p95 durations (x2 safety margin)
   - Convert sequential edges to parallel where nodes are independent
   - Add `fallback` edges for critical path nodes
   - Bump `metadata.version` patch number

7. **Show diff** of changes before writing

## Self-optimization loop

This skill enables the feedback loop:
```
Execute → Log (.osoplog) → Analyze (this skill) → Improve (.osop) → Re-execute → Better results
```

Each iteration makes the workflow more resilient and efficient.

## Epilogue (run last)

```bash
_OSOP_TEL_END=$(date +%s)
_OSOP_TEL_DUR=$(( _OSOP_TEL_END - _OSOP_TEL_START ))
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill osop-optimize --event completed --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session "$_OSOP_SESSION_ID" 2>/dev/null || true
${CLAUDE_SKILL_DIR}/../../bin/osop-telemetry-log --skill osop-optimize --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session-id "$_OSOP_SESSION_ID" 2>/dev/null &
```

Replace `OUTCOME` with `success` or `error`. If optimization suggestions were generated and applied, log a learning:
```bash
${CLAUDE_SKILL_DIR}/../../bin/osop-learnings-log '{"skill":"osop-optimize","type":"optimization","key":"SUGGESTION_KEY","insight":"WHAT_WAS_IMPROVED","confidence":7,"source":"observed"}' 2>/dev/null || true
```
