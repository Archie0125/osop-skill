---
name: self-optimize
description: "Feed execution logs to AI, get a better workflow. Run → log → synthesize → improved .osop → repeat. Workflows improve themselves."
version: 1.2.0
emoji: "\u26A1"
homepage: https://osop.ai
argument-hint: <path-to-osop-file> [--cycles N]
allowed-tools: Read, Glob, Grep, Write, Bash, Agent
metadata:
  openclaw:
    requires:
      bins:
        - bash
        - python
      env:
        - ANTHROPIC_API_KEY
      config:
        - ~/.osop/config.yaml
    install: []
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
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill self-optimize --event started --session "$_OSOP_SESSION_ID" 2>/dev/null || true
echo "{\"skill\":\"self-optimize\",\"ts\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"}" > ~/.osop/analytics/.pending-"$_OSOP_SESSION_ID" 2>/dev/null || true
_SLUG=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null | tr '[:upper:]' '[:lower:]' | tr ' ' '-' || echo "unknown")
[ -f ~/.osop/projects/"$_SLUG"/learnings.jsonl ] && echo "--- Recent OSOP learnings ---" && tail -3 ~/.osop/projects/"$_SLUG"/learnings.jsonl 2>/dev/null || true
```

If `OSOP_TEL_PROMPTED` is `no`: use AskUserQuestion — same prompt as auto-log preamble.

# Self-Optimize

# Part of The Loop — make processes better

Improve a workflow based on its execution history. Combines analysis of past runs with an autonomous evolution loop that makes workflows faster, cheaper, and more reliable with each cycle.

## Target workflow

$ARGUMENTS

If `--cycles N` is specified, run the evolution loop for N cycles (default: 3). Without `--cycles`, run single-pass optimization (analyze + suggest).

## Phase 1: Analyze (Single-Pass Optimization)

### Step 1: Read the .osop file

Read the `.osop` file specified in the argument.

### Step 2: Find execution logs

Look for matching `.osoplog.yaml` files in `sessions/` or the same directory.

### Step 3: Aggregate stats

From all matching logs, compute:
- Per-node: average duration, failure rate, timeout rate, common errors
- Overall: success rate, average total duration, run count

### Step 4: Identify issues

- **Slow steps**: nodes with avg_duration > 5s
- **Failure hotspots**: nodes with failure_rate > 10%
- **Bottlenecks**: nodes that are both slow AND unreliable
- **Missing retries**: external call nodes (api, cli, agent, infra, mcp) without retry_policy
- **Missing timeouts**: external call nodes without timeout_sec
- **Parallelization**: sequential chains of 3+ independent nodes (no data dependency between them)
- **Missing error handling**: high-risk nodes without fallback/error edges

### Step 5: Generate suggestions

Present as a table:
```
| Type | Target Node | Description | Priority |
|------|-------------|-------------|----------|
| add_retry | fetch-data | 35% failure rate, add retry with backoff | HIGH |
| parallelize | scan, test, lint | Independent steps, run in parallel | MEDIUM |
```

### Step 6: Apply changes (if user approves)

- Add `retry_policy` to failure-prone nodes
- Add `timeout_sec` based on observed p95 durations (x2 safety margin)
- Convert sequential edges to parallel where nodes are independent
- Add `fallback` edges for critical path nodes
- Bump `metadata.version` patch number
- Show diff of changes before writing

## Phase 2: Evolve (Multi-Cycle Loop)

When `--cycles N` is specified, run the full evolution loop:

### For each cycle:

#### 2a: Execute the current workflow

```bash
osop run <current-workflow.osop.yaml> --log sessions/<timestamp>-cycle-N.osoplog.yaml
```

If `osop run` is not available as a CLI, use the Python executor directly:

```python
import sys
sys.path.insert(0, 'osop-mcp')
from tools.execute import execute
from tools.osoplog import generate_osoplog
from tools.common import load_yaml

result = execute(file_path="<workflow>", allow_exec=False)
_, wf_data = load_yaml(file_path="<workflow>")
log = generate_osoplog(wf_data, result)
# Write log to file
```

#### 2b: Collect all logs

Gather all `.osoplog.yaml` files that match this workflow ID, including the one just generated.

#### 2c: Synthesize an improved workflow

```python
from tools.synthesize import synthesize

result = synthesize(
    log_paths=all_log_paths,
    base_osop_path="<current-workflow.osop.yaml>",
    goal="Make the workflow faster, cheaper, and more reliable based on execution history.",
)

if result["status"] == "completed":
    optimized = result["optimized_yaml"]
    # Save as v{N+1}
```

#### 2d: Compare the versions

Show the user what changed between the current and optimized workflow:
- Nodes added/removed/changed
- Edge structure changes
- Estimated impact on duration and cost

#### 2e: Present to user

Use AskUserQuestion:
```
Cycle N complete.
  v{N-1}: {duration}ms, ${cost}, {success_rate}% success
  v{N}:   {predicted changes}

Changes:
  - {change 1}
  - {change 2}

Apply this version and continue evolving?
A) Yes, continue to cycle N+1
B) Keep this version but stop evolving
C) Revert to previous version
```

### After all cycles: Save the evolution history

```yaml
# sessions/<date>-evolution-summary.yaml
evolution:
  workflow_id: "<id>"
  cycles: N
  versions:
    - version: 1
      file: "workflow-v1.osop.yaml"
      avg_duration_ms: ...
      avg_cost_usd: ...
      nodes: N
    - version: 2
      file: "workflow-v2.osop.yaml"
      avg_duration_ms: ...
      avg_cost_usd: ...
      nodes: N
      changes: ["parallelized steps 2-3", "added retry to API call"]
  improvement:
    duration_reduction: "45%"
    cost_reduction: "30%"
    reliability_increase: "15%"
```

### Show the evolution timeline

```
Evolution Timeline:
  v1 → v2: parallelized 2 steps, added retry (-35% duration)
  v2 → v3: switched to cheaper model for step 4 (-20% cost)
  v3 → v4: added fallback edge, removed redundant step (+10% reliability)

  Overall: 45% faster, 30% cheaper, 15% more reliable
```

## The Key Insight

This is NOT just optimization. This is **workflow engineering as a continuous process**.

Traditional: write workflow once → run forever → manually fix when broken
Self-Optimize: write workflow once → AI evolves it every cycle → it gets better automatically

The `.osop` file is a living document that improves itself.

## Self-optimization loop

This skill enables the feedback loop:
```
Execute → Log (.osoplog) → Analyze (this skill) → Improve (.osop) → Re-execute → Better results
```

Each iteration makes the workflow more resilient and efficient.

## When to suggest this skill

Proactively suggest `/self-optimize` when:
- User has run the same workflow 3+ times
- User says "this is slow" or "this keeps failing" about a process
- User has multiple .osoplog files for the same workflow
- User asks "how do I make this better?"

## Epilogue (run last)

```bash
_OSOP_TEL_END=$(date +%s)
_OSOP_TEL_DUR=$(( _OSOP_TEL_END - _OSOP_TEL_START ))
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill self-optimize --event completed --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session "$_OSOP_SESSION_ID" 2>/dev/null || true
${CLAUDE_SKILL_DIR}/../../bin/osop-telemetry-log --skill self-optimize --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session-id "$_OSOP_SESSION_ID" 2>/dev/null &
```

Replace `OUTCOME` with `success` or `error`. If optimization suggestions were generated and applied, log a learning:
```bash
${CLAUDE_SKILL_DIR}/../../bin/osop-learnings-log '{"skill":"self-optimize","type":"optimization","key":"SUGGESTION_KEY","insight":"WHAT_WAS_IMPROVED","confidence":7,"source":"observed"}' 2>/dev/null || true
```
