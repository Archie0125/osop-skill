---
name: osop-report
description: Convert .osop and .osoplog.yaml files into a standalone HTML report. Opens in any browser with dark mode, expandable nodes, and execution details.
argument-hint: <file.osop> [file.osoplog.yaml]
allowed-tools: Read, Bash, Write
---

# OSOP Report Generator

Convert workflow definition and/or execution log into a self-contained HTML report.

## Arguments

$ARGUMENTS

If no arguments provided, look for the most recent files in `sessions/` directory.

## Steps

1. **Find the files** — read the .osop file (first argument). If a .osoplog.yaml is also provided (second argument), read that too.

2. **Generate the HTML report** by running:
   ```bash
   python ${CLAUDE_SKILL_DIR}/../../scripts/osop-report.py <file.osop> [file.osoplog.yaml] -o <output.html>
   ```
   
   If Python/PyYAML is not available, generate the HTML inline using this structure:
   - Read both YAML files
   - Create a self-contained HTML with inline CSS
   - Each node becomes an expandable `<details>` element
   - Color-code by node type (orange=human, purple=agent, blue=api/cli/mcp, gray=git/docker/cicd, green=db/data)
   - Show status badges, duration bars, tool usage, AI metadata, reasoning blocks
   - Include dark mode via `prefers-color-scheme`

3. **Save the HTML** next to the source file with `-report.html` suffix.

4. **Tell the user** the file path so they can open it in a browser.

## Output format

The HTML report includes:
- Header: workflow name, status badge, duration, cost, node count
- Error banner: any failed nodes listed prominently
- Node list: expandable cards with type badge, duration bar, inputs/outputs, AI metadata, tool usage, reasoning
- Dark mode responsive, <15KB, zero external dependencies
