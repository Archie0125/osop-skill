# OSOP Policy Pack

Safety and governance policies for AI agents executing OSOP workflows. These policies apply whenever the agent interacts with workflow execution, especially in production or high-risk contexts.

## Policy 1: Dry-Run Before Production

**Rule:** Any workflow that modifies production infrastructure, deploys code, or mutates data must be executed in dry-run mode first.

**Enforcement:**
- Before calling `osop.run`, check if the workflow contains nodes with actions that target production systems (deploy, kubectl apply, database migrations, etc.).
- If production-targeting actions are detected, set `dry_run: true` on the first execution.
- Present dry-run results to the user and require explicit confirmation before live execution.

**Override:** User may say "skip dry-run" to bypass. Log the override.

## Policy 2: Approval Gates for High-Risk Operations

**Rule:** The following operation categories require an `approval` node in the workflow:

| Category | Examples |
|----------|----------|
| Production deployment | `kubectl apply`, `terraform apply`, `fly deploy` |
| Data mutation | `DROP TABLE`, `DELETE FROM`, database migrations |
| Access control changes | IAM policy updates, secret rotation |
| Financial operations | Payment processing, billing changes |
| External notifications | Mass emails, SMS broadcasts, public announcements |

**Enforcement:**
- When generating workflows, automatically insert `approval` nodes before high-risk steps.
- When validating workflows, warn if high-risk steps lack preceding approval nodes.
- Approval nodes must specify an `approvers` list — never leave it empty.

## Policy 3: Secret Handling

**Rule:** Never embed secrets, API keys, passwords, or tokens directly in OSOP workflow definitions.

**Enforcement:**
- Secrets must be referenced via environment variables: `$SECRET_NAME` or `${SECRET_NAME}`.
- When generating workflows, use variable references for any credential-like values.
- When validating, flag any string that looks like a hardcoded secret (base64 tokens, strings starting with `sk-`, `ghp_`, `xoxb-`, etc.).

## Policy 4: Scope Limitation

**Rule:** A single workflow execution must not exceed its declared scope.

**Enforcement:**
- Workflows must declare their scope in metadata: which systems they touch, what permissions they need.
- Subprocess calls must reference workflows within the same trust boundary.
- Webhook URLs must be validated against an allowlist if one is configured.

## Policy 5: Timeout and Resource Bounds

**Rule:** All long-running operations must have timeouts. All loops must have termination conditions.

**Enforcement:**
- `step` nodes with external calls should have a `timeout` field.
- `loop` nodes must have either a finite `for_each` collection or a `while` condition with a `max_iterations` safety limit.
- `retry` nodes must have `max_attempts` set (recommended maximum: 10).
- Default workflow timeout: 300 seconds. Override with `timeout_seconds` in the run command.

## Policy 6: Audit Trail

**Rule:** All workflow executions must produce an audit-friendly log.

**Enforcement:**
- Every `osop.run` call produces a structured execution log with timestamps, node IDs, inputs, outputs, and status.
- Failed nodes include error details and stack traces.
- Approval decisions are logged with the approver identity and timestamp.

## Policy 7: Rollback Readiness

**Rule:** Workflows that make changes should define rollback procedures when feasible.

**Enforcement:**
- When generating deployment workflows, suggest adding error edges that trigger rollback steps.
- Error edges from critical steps should lead to compensating actions (e.g., `kubectl rollout undo`).
- Document rollback procedures in the workflow `description` or node descriptions.
