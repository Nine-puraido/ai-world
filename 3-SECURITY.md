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

## Credential Architecture

### The Problem

If one container holds all API keys and gets hacked, everything is compromised.

### The Solution: 1Password as the Credential Layer

Instead of building custom credential proxies, we use **1Password's native
infrastructure** — Connect Server, Service Accounts, and SDKs — which already
provides scoped access, audit logging, and anomaly detection out of the box.

**One rule:** No endpoint in the system ever returns raw secret values to an
agent or worker. Secrets are resolved at runtime, used, and discarded from memory.

### How It Works (1Password Native Features)

**1. [1Password Connect Server](https://developer.1password.com/docs/connect/) — Self-Hosted Secret API**

A private REST API running inside your own VM. Two Docker containers:

```
├── [1password/connect-api]    ← REST API for fetching secrets
└── [1password/connect-sync]   ← keeps local cache in sync with 1Password cloud
```

- Runs on your infrastructure, not over public internet
- Only your services can reach it (localhost / private network)
- Caches secrets locally → low latency
- Unlimited re-requests after initial fetch

**2. [Service Accounts](https://developer.1password.com/docs/service-accounts/) — Per-Agent Scoped Tokens**

Each Master AI agent gets its own service account with access to only its vaults:

```
Service Accounts (up to 100 supported):
├── jarvis-prod      → read-only access to Dev Vault
├── jarvis-staging   → read-only access to Dev Vault (staging keys)
├── nova-prod        → read-only access to Marketing Vault
├── atlas-prod       → read-only access to Ops Vault
└── core-llm-prod    → read-only access to Core Vault (Anthropic/OpenAI keys)

Each token:
  • scoped to ONE vault only
  • read-only (can't modify secrets)
  • generates usage reports (who accessed what, when)
```

**3. [1Password SDKs](https://developer.1password.com/docs/sdks/) — Runtime Secret Resolution**

Available in Python, JavaScript, and Go. Secrets are fetched at runtime using
**secret reference URIs** — the code never contains actual credentials:

```python
# Secret reference URI — points to a secret, NOT the secret itself
facebook_key_ref = "op://Marketing-Vault/facebook-ads/api-key"

# At runtime, 1Password SDK resolves it
actual_key = client.secrets.resolve(facebook_key_ref)

# Use it for the API call
response = call_facebook_api(actual_key, ad_data)

# actual_key is garbage collected — never written to disk
```

**4. [Events API](https://developer.1password.com/docs/events-api/audit-events/) — Audit + Anomaly Detection**

```
Built-in:
  • 365 days of audit logs
  • tracks every secret access (who, what, when)
  • token creation / revocation tracking
  • failed access attempts logged

Integrates with:
  • Datadog (pre-built 1Password detection rules)
  • Splunk (sign-in attempts, item usage, audit events)
  • Panther (pre-built anomaly detections)
  • or poll the Events API directly from our own alerting
```

**5. [Agentic AI Pattern](https://developer.1password.com/docs/sdks/ai-agent/) — Official 1Password Guide**

1Password has an official tutorial for exactly our use case — AI agents that
need credentials at runtime without ever storing them. The recommended pattern:

```
1. Store credentials in a dedicated vault scoped for the task
2. Create a service account token with read-only access to that vault
3. Fetch credentials at runtime using secret reference URIs via SDK
4. Pass secrets to execution logic without hardcoding or exposing
5. Update secrets centrally without touching agent logic (dynamic rotation)
```

### Container Map (Production)

```
One VM (24/7)
│
├── [Router]                      ← message routing only
│     • receives your messages from API Server
│     • routes to the correct agent container
│     • NO brain, NO tokens, NO secrets
│     • tiny attack surface — no credentials to steal
│
├── [Jarvis Agent Container]      ← isolated agent process
│     • Planner + Policy Engine + Broker + Verifier
│     • own 1Password Service Account (jarvis-prod → Dev Vault only)
│     • resolves secrets via 1Password SDK at runtime
│     • secrets live in memory briefly, never on disk
│
├── [Nova Agent Container]        ← isolated agent process
│     • own 1Password Service Account (nova-prod → Marketing Vault only)
│
├── [Atlas Agent Container]       ← isolated agent process
│     • own 1Password Service Account (atlas-prod → Ops Vault only)
│
├── [Agent Manager]               ← create/edit/delete agent containers
│     • CRITICAL action — only human from Dashboard can trigger
│     • no agent can create another agent
│
├── [1Password Connect API]       ← private secret REST API (1Password official)
├── [1Password Connect Sync]      ← keeps secrets in sync with 1Password cloud
│
├── [Worker Manager]              ← Docker lifecycle only (shared across all agents)
│     • spawns/kills worker containers
│     • watches heartbeats
│     • no API keys, no LLM
│
├── [Redis]                       ← message transport
├── [PostgreSQL]                  ← state + audit log
├── [Dashboard (Next.js)]         ← UI (includes Agent Management page)
│
└── [Ephemeral Workers]           ← spawned per task, destroyed after
      • no API keys
      • no 1Password access
      • no direct external API access
      • request actions → agent container handles with scoped secrets
```

### Credential Flow (Example: Push Code to GitHub)

```
Worker: "push this code to repo X"
  │
  ▼
Router:
  │ looks up task-id → belongs to Jarvis
  │ forwards message to Jarvis Agent Container
  │ (Router never sees secrets — just routes)
  │
  ▼
Jarvis Agent Container:
  │ policy check: is this worker allowed to push? ✓
  │ uses jarvis-prod Service Account (only this container has it)
  │ calls: client.secrets.resolve("op://Dev-Vault/github/token")
  │
  ▼
1Password Connect (on same VM):
  │ checks: jarvis-prod → Dev Vault → allowed ✓
  │ returns GitHub token
  │
  ▼
Jarvis Agent Container:
  │ calls GitHub API with token
  │ token garbage collected from memory
  │ returns to Worker: "pushed, commit abc123"
  │
  ▼
Worker: "push done" (never saw the token)
```

### What Each Hack Gets You

```
Router hacked:
  ✗ secrets                     → Router has NO tokens, NO 1Password access
  ✗ LLM access                 → Router has NO API keys
  ~ can reroute messages        → but agent containers validate message signatures
  → kill Router = agents stop receiving messages, no secrets exposed
  → blast radius: message routing disrupted, zero credential exposure

Single agent container hacked (e.g. Jarvis):
  ✗ raw API keys on disk        → secrets are never written to disk
  ✗ other agents' vaults        → Jarvis can only access Dev Vault, not Marketing/Ops
  ~ secrets in memory briefly   → attacker must catch the ~100ms window
  ✓ can request actions         → but policy + rate limits still apply
  → kill Jarvis container = Jarvis's access dies, other agents unaffected
  → blast radius: ONE agent's vault only (not all vaults)

1Password Connect hacked:
  ~ can serve secrets           → but only to authenticated Service Accounts
  ✗ stores secrets in plaintext → cache is encrypted
  → revoke Connect credentials = cut all access instantly

Worker hacked:
  ✗ nothing                     → no keys, no 1Password access, no tokens
  → container dies after task anyway
```

### 1Password Vault Layout

Each agent runs in its own container with its own 1Password Service Account.
A compromised Jarvis container cannot access Nova's or Atlas's vault.

```
1Password Vaults (one per domain):
├── Dev Vault          ← GitHub, npm, Vercel tokens
├── Marketing Vault    ← Facebook, Google Ads, Mailchimp tokens
├── Ops Vault          ← AWS, Slack, monitoring tokens
├── Infra Vault        ← AWS, Docker, SSH, monitoring tokens
└── Core Vault         ← Anthropic, OpenAI keys (for LLM API calls)

Access matrix (each column = a separate container with its own Service Account):
  Vault              │ jarvis      │ nova        │ atlas       │ sentinel    │ router │ workers
                     │ container   │ container   │ container   │ container   │        │
  ───────────────────┼─────────────┼─────────────┼─────────────┼─────────────┼────────┼────────
  Dev Vault          │ read        │  -          │   -         │   -         │   -    │   -
  Marketing Vault    │   -         │ read        │   -         │   -         │   -    │   -
  Ops Vault          │   -         │  -          │ read        │   -         │   -    │   -
  Infra Vault        │   -         │  -          │   -         │ read        │   -    │   -
  Core Vault         │ read        │ read        │ read        │ read        │   -    │   -
  (LLM keys)        │             │             │             │             │        │
```

### Key Principles

1. **No raw secret retrieval** — never expose `get_secret(name)` to any worker
   or LLM path. Workers request **actions** ("push code", "post ad"), the agent's
   container resolves secrets internally via SDK and executes.
2. **Separate tokens aggressively** — one 1Password Service Account per agent per
   environment. Jarvis can't read Marketing Vault. Nova can't read Dev Vault.
3. **Secrets never on disk** — resolved at runtime via `secrets.resolve()`, used
   for the API call, then garbage collected. Never written to `.env` or files.
4. **Short-lived permissions** — per-task capability tokens with TTL (minutes),
   scope, and rate limits. Even a compromised orchestrator can only do limited
   actions briefly.
5. **Assume breach will happen** — 1Password Events API + SIEM integration for
   detection. Pre-built breach response playbook for fast recovery.

### What We Don't Need to Build (1Password Provides It)

| Planned custom feature | 1Password native replacement |
|---|---|
| Custom Action Broker containers | 1Password Connect Server (2 Docker containers) |
| Per-agent scoped credential access | Service Accounts (up to 100, vault-scoped) |
| Fetch-use-delete credential flow | SDK `secrets.resolve()` with secret reference URIs |
| Audit logging for secret access | Events API (365 days retention) |
| Anomaly detection | Datadog / Splunk / Panther integrations |
| Token revocation | Built-in, 1 click in 1Password dashboard |
| Dynamic secret rotation | Update centrally, agents pick up changes automatically |

---

## What's Built In (Day 1)

These are non-negotiable. Simple to implement, high impact.

| Control | How |
|---|---|
| No raw shell | Workers get specific tool functions only (`write_file`, `run_test`, `git_commit`). No `shell()` or `exec()` exists |
| Action-based tool proxy | Workers never hold API keys. They request actions → agent container resolves secrets via 1Password SDK → executes → returns result only |
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

## Breach Response Playbook

Pre-defined steps. Practice this once — speed matters more than perfect prevention.

```
BREACH DETECTED (1Password Events API anomaly / SIEM alert / manual discovery)

Step 1: PAUSE          pause all workers for the affected agent        (~seconds)
Step 2: REVOKE         revoke affected agent's Service Account token   (~1 click)
                       in 1Password dashboard
Step 3: ISOLATE        disconnect the compromised agent container      (~seconds)
                       from the network (other agents keep running)
Step 4: ASSESS         check 1Password Events API: what was accessed?  (~minutes)
                       check PostgreSQL audit log: what actions ran?
                       scope is limited — only that agent's vault was exposed
Step 5: ROTATE         rotate affected downstream API keys             (~minutes)
                       (update in 1Password vault → agent picks up automatically)
Step 6: REISSUE        create new Service Account token for that agent (~minutes)
Step 7: RESUME         restart the agent container with new token      (~minutes)
                       other agents were never affected

Total recovery: ~10-15 minutes if practiced

Key advantage: only the breached agent's container needs isolation.
Other agents continue operating normally. Blast radius = one vault.
Secrets updated in 1Password vault are picked up automatically.
No need to redeploy all containers or update .env files.
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
- *Proposed:* upgrade to **mTLS** (mutual TLS) between Router, each agent
  container, 1Password Connect, Worker Manager, and Redis gateway.
  Each agent container gets its own TLS certificate. Router validates
  agent identity before forwarding messages. Per-service RBAC so
  workers literally cannot publish on agent channels, and one agent
  container cannot impersonate another.
  Trade-off: certificate management complexity

### 3. Secret rotation

1Password handles central secret storage, but downstream API keys (GitHub,
Facebook, etc.) are still long-lived.

- *Concern:* if a key leaks during the runtime window, it stays valid
- *Proposed:* automatic rotation on a schedule. For AWS/GCP/Azure, use
  **workload identity / OIDC** instead of long-lived static keys. Keep
  1Password mainly for SaaS keys that cannot be made ephemeral.
  Trade-off: rotation logic adds complexity per integration

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

- *Current mitigation:* PostgreSQL with append-only application logic.
  1Password Events API provides a separate tamper-resistant audit trail for
  secret access (365 days retention).
- *Proposed:* write-once storage (S3 with Object Lock, or a dedicated
  append-only log service) for our own application logs. Combine with
  1Password Events API for complete picture. Trade-off: additional infrastructure cost

---

## Security Roadmap

| Phase | Security Level | What's in place |
|---|---|---|
| Prototype (Phase 1-3) | Dev-safe | No shell, file isolation, policy checks, signed messages, 1Password SDK for secrets |
| Supervised autonomy (Phase 4-7) | Work-hours safe | + network egress control, lockfile enforcement, scope checks, Verifier gates, 1Password Connect Server, per-agent Service Accounts, Events API monitoring |
| 24/7 autonomy (Phase 8+) | Production-safe | + mTLS, auto secret rotation, gVisor/Firecracker, private registry, taint tracking, immutable logs, SIEM integration (Datadog/Panther) |

> **Bottom line:** the prototype is safe for development with you watching.
> 24/7 unsupervised autonomy requires the upgrades in the roadmap above.
> We'll revisit each "Proposed" item when we reach that phase.
