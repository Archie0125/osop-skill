---
name: api-sop-gen
description: "Paste API docs, get visual SOP workflows. Converts markdown API documentation into .osop files with step-by-step flows."
version: 1.0.0
emoji: "\U0001F504"
homepage: https://osop.ai
argument-hint: [API documentation URL or paste inline]
allowed-tools: Read, Glob, Grep, Write, Bash
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
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill api-sop-gen --event started --session "$_OSOP_SESSION_ID" 2>/dev/null || true
echo "{\"skill\":\"api-sop-gen\",\"ts\":\"$(date -u +%Y-%m-%dT%H:%M:%SZ)\"}" > ~/.osop/analytics/.pending-"$_OSOP_SESSION_ID" 2>/dev/null || true
_SLUG=$(basename "$(git rev-parse --show-toplevel 2>/dev/null)" 2>/dev/null | tr '[:upper:]' '[:lower:]' | tr ' ' '-' || echo "unknown")
[ -f ~/.osop/projects/"$_SLUG"/learnings.jsonl ] && echo "--- Recent OSOP learnings ---" && tail -3 ~/.osop/projects/"$_SLUG"/learnings.jsonl 2>/dev/null || true
```

If `OSOP_TEL_PROMPTED` is `no`: use AskUserQuestion — same prompt as sop-doc preamble.

# API SOP Generator

# Part of SOP Doc — understand how things work

Convert API documentation into visual .osop workflow files. Takes markdown, OpenAPI specs, or plain text API docs and produces structured step-by-step OSOP workflows.

## Arguments

$ARGUMENTS

If no arguments provided, ask the user to paste or provide a path to their API documentation.

## What to do

### Step 1: Read the API documentation

Read the user's API documentation. Supported input formats:
- **Markdown** — API reference docs with endpoints, parameters, examples
- **OpenAPI/Swagger** — JSON or YAML spec files (v2 or v3)
- **Plain text** — informal descriptions of API flows
- **URL** — fetch documentation from a web URL

If the input is a file path, read it. If it's pasted inline, parse it directly.

### Step 2: Parse operations and sequences

Identify the logical operations in the API:
- **Authentication flows** — API key setup, OAuth token exchange, refresh cycles
- **CRUD sequences** — create → read → update → delete patterns
- **Multi-step workflows** — operations that depend on previous responses (e.g., create user → get user ID → assign role)
- **Webhook/event flows** — register webhook → receive event → process → acknowledge
- **Pagination patterns** — initial request → check next page → loop

For each operation sequence, determine:
- Required inputs (headers, body, query params)
- Expected outputs (response fields used by later steps)
- Error handling (status codes, retry logic)
- Data dependencies between steps

### Step 3: Generate .osop YAML

For each identified workflow, generate a valid `.osop` file:

```yaml
osop_version: "1.0"
id: "api-<service-name>-<flow-name>"
name: "<Service> — <Flow Description>"
description: "<What this API flow accomplishes>"
version: "1.0.0"
tags: [api, <service-name>, <domain>]

runtime:
  env:
    - name: "<SERVICE>_API_KEY"
      description: "API key for authentication"
    - name: "<SERVICE>_BASE_URL"
      description: "Base URL for the API"
      default: "https://api.example.com/v1"

nodes:
  - id: "auth"
    type: "api"
    subtype: "rest"
    name: "Authenticate"
    description: "Obtain access token"
    config:
      method: "POST"
      url: "${<SERVICE>_BASE_URL}/auth/token"
      headers:
        Authorization: "Bearer ${<SERVICE>_API_KEY}"
    security:
      risk_level: "medium"
      secrets:
        - "<SERVICE>_API_KEY"

  - id: "<operation-id>"
    type: "api"
    subtype: "rest"
    name: "<Operation Name>"
    description: "<What this step does>"
    config:
      method: "<GET|POST|PUT|DELETE>"
      url: "${<SERVICE>_BASE_URL}/<path>"
      headers:
        Authorization: "Bearer ${auth.response.token}"
    timeout_sec: 30
    retry_policy:
      max_retries: 3
      backoff: "exponential"

edges:
  - from: "auth"
    to: "<first-operation>"
    mode: "sequential"
  - from: "<operation-a>"
    to: "<operation-b>"
    mode: "sequential"
    label: "Pass response data"
```

Guidelines for generation:
- Each API call becomes an `api` node with `subtype: rest`
- Include `config` with method, url, headers, body where applicable
- Add `timeout_sec` to all API nodes (default: 30s)
- Add `retry_policy` for idempotent operations (GET, PUT)
- Mark authentication nodes with `security.secrets`
- Use `conditional` edges for branching logic (e.g., different paths based on response status)
- Use `loop` edges for pagination
- Use `parallel` edges for independent operations

### Step 4: Validate the output

Verify the generated .osop:
- All node IDs are unique and valid (lowercase, hyphens only)
- All edges reference existing node IDs
- No orphan nodes (every node has at least one edge)
- Required fields are present (osop_version, id, name, nodes, edges)
- Security metadata is set for nodes handling secrets

### Step 5: Save and direct the user

1. Save the generated `.osop` file(s) — suggest a path like `workflows/<service>-<flow>.osop.yaml`
2. Tell the user they can visualize the workflow at https://osop.ai/sop-doc or the OSOP visual editor at https://osop-editor.vercel.app
3. If multiple flows were generated, list them all with brief descriptions

## Example output

For a Stripe payment API doc, you might generate:
- `workflows/stripe-payment-flow.osop.yaml` — Create customer → Create payment intent → Confirm payment → Handle webhook
- `workflows/stripe-subscription-flow.osop.yaml` — Create customer → Create subscription → Handle invoice webhook → Handle payment failure

## Epilogue (run last)

```bash
_OSOP_TEL_END=$(date +%s)
_OSOP_TEL_DUR=$(( _OSOP_TEL_END - _OSOP_TEL_START ))
${CLAUDE_SKILL_DIR}/../../bin/osop-timeline-log --skill api-sop-gen --event completed --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session "$_OSOP_SESSION_ID" 2>/dev/null || true
${CLAUDE_SKILL_DIR}/../../bin/osop-telemetry-log --skill api-sop-gen --duration "$_OSOP_TEL_DUR" --outcome "OUTCOME" --session-id "$_OSOP_SESSION_ID" 2>/dev/null &
```

Replace `OUTCOME` with `success` or `error`.
