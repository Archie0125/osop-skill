---
name: sop-diff
description: "Compare two workflow runs or versions. See what changed — duration, cost, status per step. Like git diff for processes."
version: 1.0.0
emoji: "\U0001F4CA"
homepage: https://osop.ai
argument-hint: <file-a.osop|.osoplog> <file-b.osop|.osoplog>
allowed-tools: Read, Glob, Grep, Bash
metadata:
  openclaw:
    requires:
      bins:
        - bash
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
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill sop-diff --event started --session "$_OSOP_SESSION_ID" 2>/dev/null || true
echo "{\"skill\":\"sop-diff\",\"ts\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"}" > ~/.osop/analytics/.pending-"$_OSOP_SESSION_ID" 2>/dev/null || true
_SLUG=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null | tr '[:upper:]' '[:lower:]' | tr ' ' '-' || echo "unknown")
[ -f ~/.osop/projects/"$_SLUG"/learnings.jsonl ] && echo "--- Recent OSOP learnings ---" && tail -3 ~/.osop/projects/"$_SLUG"/learnings.jsonl 2>/dev/null || true
```

If `OSOP_TEL_PROMPTED` is `no`: use AskUserQuestion — same prompt as sop-doc preamble.

# SOP Diff

# Part of SOP Doc — understand how things work

Compare two .osop workflow definitions or two .osoplog execution records side by side. Shows structural changes, performance deltas, and status differences — like `git diff` for processes.

## Arguments

$ARGUMENTS

Expects two file paths. Both can be `.osop` files (version comparison) or both `.osoplog.yaml` files (run comparison), or one of each (definition vs execution comparison).

## What to do

### Step 1: Read both files

Read File A and File B. Determine the comparison type:
- **Version diff** — two `.osop` files (comparing workflow structure)
- **Run diff** — two `.osoplog.yaml` files (comparing execution results)
- **Spec vs reality** — one `.osop` + one `.osoplog.yaml` (what was planned vs what happened)

### Step 2: Compare structure

#### For .osop vs .osop (Version Diff)

Compare nodes and edges:

| Aspect | File A | File B | Change |
|--------|--------|--------|--------|
| **Nodes added** | — | list new node IDs | + |
| **Nodes removed** | list removed node IDs | — | - |
| **Nodes modified** | old config | new config | ~ |
| **Edges added** | — | new connections | + |
| **Edges removed** | old connections | — | - |
| **Edge mode changes** | old mode | new mode | ~ |

Also compare:
- Metadata changes (version, tags, description)
- Runtime config changes (env vars, requirements)
- Security changes (risk levels, approval gates, permissions)

#### For .osoplog vs .osoplog (Run Diff)

Compare execution metrics per node:

| Node | Metric | Run A | Run B | Delta |
|------|--------|-------|-------|-------|
| fetch-data | duration_ms | 1200 | 800 | -33% |
| fetch-data | status | COMPLETED | COMPLETED | = |
| process | duration_ms | 3500 | 2100 | -40% |
| deploy | status | FAILED | COMPLETED | fixed |

Also compare:
- **Overall status**: both succeeded? one failed?
- **Total duration**: faster or slower?
- **Total cost**: cheaper or more expensive?
- **Tool usage**: different tools or call counts?
- **Failure points**: which nodes failed in each run?

#### For .osop vs .osoplog (Spec vs Reality)

Compare what was defined vs what actually happened:

| Node | Defined | Actual | Gap |
|------|---------|--------|-----|
| fetch-data | timeout: 30s | duration: 45s | exceeded timeout |
| process | type: agent | tools: [Read, Bash] | as expected |
| deploy | risk: high | status: FAILED | risk materialized |

Flag:
- Nodes defined but never executed (skipped)
- Nodes executed but not in the definition (unexpected)
- Timeout violations
- Risk predictions vs actual outcomes

### Step 3: Present the comparison

Output a formatted comparison report:

```
=== SOP Diff: <File A> vs <File B> ===
Type: <Version Diff | Run Diff | Spec vs Reality>

Summary:
  Nodes: +N added, -N removed, ~N modified
  Edges: +N added, -N removed, ~N changed
  Duration: A=Xms → B=Yms (Z% change)
  Cost: A=$X → B=$Y (Z% change)
  Status: A=STATUS → B=STATUS

Details:
  [detailed table as above]

Key Findings:
  - [most important change 1]
  - [most important change 2]
  - [most important change 3]
```

### Step 4: Suggest actions (if applicable)

Based on the diff, suggest:
- If a run got slower: investigate the slow nodes, suggest optimization
- If a run fixed a failure: note what changed and why it worked
- If structure changed: highlight whether the change simplified or complicated the workflow
- If spec vs reality diverged: suggest updating the spec or adding constraints

## Epilogue (run last)

```bash
_OSOP_TEL_END=$(date +%s)
_OSOP_TEL_DUR=$(( _OSOP_TEL_END - _OSOP_TEL_START ))
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill sop-diff --event completed --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session "$_OSOP_SESSION_ID" 2>/dev/null || true
${CLAUDE_SKILL_DIR}/../../bin/osop-telemetry-log --skill sop-diff --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session-id "$_OSOP_SESSION_ID" 2>/dev/null &
```

Replace `OUTCOME` with `success` or `error`.
