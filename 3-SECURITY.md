# AI World — Security

> **Read [1-ARCHITECTURE.md](1-ARCHITECTURE.md) first.** This doc covers how the system stays safe —
> what's built in from day 1, what needs future work, and the security roadmap.

---

## Core Rule

**Containers alone do not stop prompt injection.**

Prompt injection happens at the LLM level — the LLM gets tricked into making
bad tool calls. The container doesn't help because the tool calls look "legitimate"
from the container's perspective.

Prompt injection is stopped by:
tool-level policy checks + signed identity + network egress control + treating
all external data as untrusted input (never as instructions).

---

## What's Built In (Day 1)

These are non-negotiable. Simple to implement, high impact.

| Control | How |
|---|---|
| No raw shell | Workers get specific tool functions only (`write_file`, `run_test`, `git_commit`). No `shell()` or `exec()` exists |
| Tool proxy for secrets | Workers never hold API keys. They request actions → Orchestrator executes with real credentials → returns result |
| Policy validates every tool call | Every tool call checks the capability lease + policy rules before executing. LLM reasoning is irrelevant — only the action matters |
| Signed messages | Every Redis message envelope includes an HMAC signature. Workers can only publish `task.result`, `heartbeat`, `question`. They cannot publish `permission.granted` or `task.assign` |
| File system isolation | Each worker's container mounts only its own `/workspace/task-{id}/`. Other tasks and host filesystem are invisible |
| Network egress deny-by-default | Workers can only reach whitelisted domains (npm/pypi registries) through a forward proxy. All other outbound traffic is blocked |
| Block cloud metadata | `169.254.169.254` is blocked to prevent cloud credential theft |
| No postinstall scripts | `npm install --ignore-scripts`. Lockfile required for reproducibility |
| Ephemeral workers | Container destroyed after task. Even if compromised, the attacker loses access immediately |
| Scope check in Verifier | Rejects any diff that touches files outside the task's assigned scope |

---

## Risk Matrix (Single Source of Truth)

One file. All components reference it. No duplicates.

```yaml
risk_matrix:
  LOW:
    auto_approve: always
    actions: [read_files, write_to_task_branch, run_tests, linter, git_commit]
    token_limit: 10000
    cost_limit: 0.50

  MEDIUM:
    auto_approve: work_hours_only
    queue_outside_hours: true
    actions: [pkg_install, create_branch, external_api_read, web_research]
    token_limit: 50000
    cost_limit: 2.00

  HIGH:
    auto_approve: never
    requires: planner_approval
    actions: [deploy_staging, modify_ci, change_db_schema, external_api_write]
    token_limit: 100000
    cost_limit: 5.00

  CRITICAL:
    requires: human_approval
    actions: [deploy_production, delete_data, manage_secrets, spend_over_50]
    notification: immediate
```

---

## Guardrails (Overnight Safety)

Hard limits that prevent disasters when you're sleeping.

```yaml
guardrails:
  budget:
    daily_token_limit: 500000
    daily_dollar_limit: 10.00
    per_task_token_limit: 100000
    per_task_dollar_limit: 2.00

  retries:
    max_per_task: 3
    max_per_hour: 10
    backoff: exponential

  time_windows:
    auto_mode: "08:00-22:00"        # full autonomy (LOW + MEDIUM)
    queue_mode: "22:00-08:00"       # HIGH+ tasks queued for morning

  kill_switch:
    triggers:
      - 3 consecutive failures
      - daily budget exceeded
      - unknown error type
    action: pause all workers, notify human via Telegram / Dashboard
```

---

## Known Concerns (Needs Future Discussion)

These are real risks that are acceptable during prototype/development but
must be resolved before running 24/7 autonomous production workloads.

### 1. Worker-Manager blast radius

The Worker Manager can spawn/kill any worker container and access all mounted
workspaces. If compromised, it controls everything.

- *Current mitigation:* rootless Docker, non-root user, no-new-privileges
- *Concern:* rootless Docker is still powerful enough to mount filesystems
  and control all worker containers
- *Proposed:* evaluate **gVisor** or **Firecracker** micro-VMs for stronger
  isolation. Trade-off: more complex setup, higher resource overhead

### 2. Service identity (mTLS)

Currently, services authenticate via signed message envelopes (HMAC).
This is good for prototype but has limits — shared HMAC keys can leak.

- *Concern:* if a worker extracts the signing key, it can impersonate any service
- *Proposed:* upgrade to **mTLS** (mutual TLS) between orchestrator,
  worker-manager, and Redis gateway. Per-service RBAC so workers literally
  cannot publish on orchestrator channels. Trade-off: certificate management complexity

### 3. Secret rotation

API keys and tokens are currently static (set once in Docker secrets/Vault).

- *Concern:* if a key leaks, it stays valid until manually rotated
- *Proposed:* automatic rotation on a schedule. Short-lived scoped tokens
  for workers (expire in minutes, not days). Secret redaction in all logs.
  Trade-off: rotation logic adds complexity

### 4. Supply chain attacks (npm/pip packages)

Workers can install packages from public registries. Malicious packages exist.

- *Current mitigation:* `--ignore-scripts`, lockfile required, Policy approval
- *Concern:* a dependency of a dependency could still contain malicious code
  that runs at import time (not just postinstall)
- *Proposed:* private registry proxy (Verdaccio/Artifactory) that only allows
  pre-approved packages. Or: pre-baked worker images with common packages
  already installed, no runtime installs at all. Trade-off: maintenance burden

### 5. Prompt injection via legitimate tools

Blocking `shell()` helps, but `write_file()` + `git_commit()` is still
powerful. A prompt-injected worker could write malicious code and commit it.

- *Current mitigation:* Verifier security scan (semgrep) + scope check +
  different LLM review
- *Concern:* sophisticated injections could write code that passes static
  analysis but behaves maliciously at runtime
- *Proposed:* taint tracking — mark all external inputs as "untrusted data"
  that cannot directly trigger privileged tool calls. Trade-off: complex to implement

### 6. Immutable audit log

Current audit log is in PostgreSQL. If the database is compromised,
logs can be altered to hide an attack.

- *Current mitigation:* PostgreSQL with append-only application logic
- *Proposed:* write-once storage (S3 with Object Lock, or a dedicated
  append-only log service). Anomaly detection alerts on unusual patterns.
  Trade-off: additional infrastructure cost

---

## Security Roadmap

| Phase | Security Level | What's in place |
|---|---|---|
| Prototype (Phase 1-3) | Dev-safe | No shell, file isolation, policy checks, tool proxy, signed messages |
| Supervised autonomy (Phase 4-7) | Work-hours safe | + network egress control, lockfile enforcement, scope checks, Verifier gates |
| 24/7 autonomy (Phase 8+) | Production-safe | + mTLS, secret rotation, gVisor/Firecracker, private registry, taint tracking, immutable logs |

> **Bottom line:** the prototype is safe for development with you watching.
> 24/7 unsupervised autonomy requires the upgrades in the roadmap above.
> We'll revisit each "Proposed" item when we reach that phase.
