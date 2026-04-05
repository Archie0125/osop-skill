---
name: osop-evolve
description: "Self-optimizing workflow loop: run → log → AI synthesize → improved .osop → run again. Continuous improvement."
version: 1.0.0
emoji: "\U0001F504"
homepage: https://osop.ai
argument-hint: <workflow.osop.yaml> [--cycles N]
allowed-tools: Read, Glob, Grep, Write, Bash, Agent
metadata:
  openclaw:
    requires:
      bins:
        - bash
        - python
      env:
        - ANTHROPIC_API_KEY
    install: []
    always: false
user-invocable: true
disable-model-invocation: false
---

# OSOP Evolve — Self-Optimizing Workflow Loop

The autonomous improvement loop:

```
Execute .osop → Generate .osoplog → AI analyzes logs → Produces improved .osop → Repeat
```

Each cycle the workflow gets faster, cheaper, more reliable. You just watch.

## Arguments

`$ARGUMENTS` should be a path to a `.osop.yaml` workflow file. Optional `--cycles N` (default: 3).

## What to do

### Step 1: Load the workflow

Read the `.osop.yaml` file specified in `$ARGUMENTS`. Parse the path and any flags.

```bash
# Find all existing .osoplog files for this workflow
WORKFLOW_FILE="$1"
WORKFLOW_ID=$(python -c "import yaml; print(yaml.safe_load(open('$WORKFLOW_FILE', encoding='utf-8')).get('id','unknown'))" 2>/dev/null)
echo "Workflow: $WORKFLOW_ID"

# Find existing logs
find sessions/ . -name "*.osoplog.yaml" -exec grep -l "$WORKFLOW_ID" {} \; 2>/dev/null | head -20
```

### Step 2: Run the evolution loop

For each cycle (default 3):

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

#### 2b: Collect all logs for this workflow

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
    # Write the improved .osop
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

### Step 3: Save the evolution history

After all cycles, save a summary:

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

### Step 4: Show the evolution timeline

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
OSOP Evolve: write workflow once → AI evolves it every cycle → it gets better automatically

The `.osop` file is a living document that improves itself.

## When to suggest this skill

Proactively suggest `/osop-evolve` when:
- User has run the same workflow 3+ times
- User says "this is slow" or "this keeps failing" about a process
- User has multiple .osoplog files for the same workflow
- User asks "how do I make this better?"

## Epilogue (run last)

```bash
_OSOP_TEL_END=$(date +%s)
_OSOP_TEL_DUR=$(( _OSOP_TEL_END - _OSOP_TEL_START ))
echo "Evolution completed in ${_OSOP_TEL_DUR}s"
```
