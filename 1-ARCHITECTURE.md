# AI World

## Reading Order

```
1-ARCHITECTURE.md       ← you are here (the big picture)
2-WORKFLOW.md            ← how tasks flow step by step
3-SECURITY.md            ← how it stays safe
4-DASHBOARD-SPEC.md      ← what the UI looks like
5-TECHNICAL-DETAILS.md   ← protocol, code, DB schema (for building)
6-SETUP-GUIDE.md         ← when you're ready to start
```

---

## What Is This?

AI agents today can code, research, and reason on their own.
But they still need a human in the loop for every decision —
and that human is the bottleneck.

**AI World replaces that human with a Master AI Agent.**

Sub-agents do the work. They're already skilled. The Master AI doesn't
code — it plans, decides, reviews, and corrects. A second brain from
a **different AI provider**, so no single model's blind spots go unchecked.

You're the CEO. You have a group of Master AI Agents — Jarvis for software,
Nova for marketing, Atlas for operations. Each Master AI can create, hire,
and set up its own group of employees (sub-agents) for every task.
You give goals. They run their teams. You approve what matters.

> **Terminology:**
> - **Master AI Agent** (aka Manager) = Jarvis, Nova, Atlas — the brain that plans and decides
> - **Sub-agents** (aka Workers) = temporary employees that do the actual work

---

## How It Works (30 second version)

```
You: "Build me a todo app with auth and deploy it"
│
▼
JARVIS (Manager) receives your goal
│
├── 1. PLANS: Breaks goal into 5 subtasks + dependency graph
│      Picks tech stack on its own: Next.js, PostgreSQL, JWT
│
│      task-1: DB schema         (no dependency — can start now)
│      task-2: REST API          (depends on task-1)
│      task-3: Frontend          (depends on task-2)
│      task-4: Tests             (depends on task-2 + task-3)
│      task-5: Deploy            (depends on all — needs human OK)
│
│
├── 2. WAVE 1 — tasks with no dependencies run PARALLEL
│
│      ┌─── SUB-AGENT A (task-1: DB) ─────────────────────┐
│      │ Writes schema, migrations                         │
│      │ ✅ Done. Returns result.                           │
│      └──────────────────────────────────────────────────┘
│      (only 1 task has no dependency in this example,
│       but if task-1 and task-2 were independent,
│       both sub-agents would run at the same time)
│
├── 3. REVIEWS: Jarvis checks task-1 output
│      "Good, but missing an index on user_id."
│      → Follow-up prompt → sub-agent fixes → Jarvis approves
│      → Verifier gates (tests, security scan) + different AI → ✅ merged
│
│
├── 4. WAVE 2 — task-1 done, so task-2 is unblocked
│
│      ┌─── SUB-AGENT B (task-2: API) ────────────────────┐
│      │ Writes API code                                   │
│      │ ❓ "DB or cookies for refresh tokens?"             │
│      │ Jarvis answers: "DB. We need revocation."         │
│      │ Sub-agent continues. Finishes.                    │
│      └──────────────────────────────────────────────────┘
│
│      Jarvis reviews → "Missing CORS" → follow-up → fixed
│      Verifier → ✅ merged
│
│
├── 5. WAVE 3 — task-2 done, so task-3 + task-4 BOTH unblocked
│      TWO sub-agents run IN PARALLEL:
│
│      ┌─── SUB-AGENT C (task-3: Frontend) ───┐  ┌─── SUB-AGENT D (task-4: Tests) ───┐
│      │ Builds React UI                       │  │ Writes API + frontend tests        │
│      │ Asks Jarvis: "Add shadcn/ui?"         │  │ Runs test suite                    │
│      │ Jarvis: "Yes."                        │  │ ✅ 24 tests, 87% coverage           │
│      │ ✅ Done.                               │  │ ✅ Done.                             │
│      └──────────────────────────────────────┘  └──────────────────────────────────────┘
│                    │                                          │
│                    └──── both reviewed + verified ────────────┘
│
│
├── 6. ALL CODE DONE. Asks you to approve deploy.
│      You: "Approve" (one tap on Telegram / Dashboard / CLI)
│
│      ┌─── SUB-AGENT E (task-5: Deploy) ─────────────────┐
│      │ Deploys frontend to Vercel, API to Railway        │
│      │ Health checks pass.                               │
│      └──────────────────────────────────────────────────┘
│
└── 7. Done. Reports back.

Total: 5 tasks │ 3 waves │ 32 minutes │ $2.84 │ you typed 2 messages
```

**Parallel, not single-threaded.** Jarvis builds a dependency graph and runs
independent tasks at the same time. Tasks that depend on others wait until
their dependencies are done, then launch immediately.

```
WAVE 1:  [task-1 DB]                          ← 1 sub-agent
WAVE 2:  [task-2 API]                         ← 1 sub-agent (waited for task-1)
WAVE 3:  [task-3 Frontend] + [task-4 Tests]   ← 2 sub-agents IN PARALLEL
WAVE 4:  [task-5 Deploy]                      ← 1 sub-agent (waited for all)
```

**The core loop per task:**
```
Jarvis assigns prompt → Sub-agent works (asks Jarvis if confused)
  → Jarvis reviews → Good? Ship it. Not good? Follow-up prompt. Repeat.
  → Verifier checks → merged → next wave of tasks launches.
```

> **Full detailed walkthrough:** see [2-WORKFLOW.md](2-WORKFLOW.md)

---

## The Architecture

### Core Principle

**AI controls decisions. Hard policy controls power.**

No single component has full authority. The system has checks and balances —
like a government, not a dictatorship.

```
You (goals) → Planner → Policy → Capability Lease → Worker → Verifier → Result
```

### 1Password AI Agent Support

1Password has a first-party **Agentic AI pattern** built for exactly this use case.
We don't need to build custom credential proxies — 1Password provides the entire
credential layer out of the box:

- **Connect Server** — a private REST API running on your VM. Two Docker containers
  (`connect-api` + `connect-sync`). Secrets fetched locally, not over public internet.
- **Service Accounts** — up to 100 scoped tokens. Each agent gets its own, locked to
  one vault, read-only. Jarvis can't read Marketing Vault. Nova can't read Dev Vault.
- **SDKs** (Python, JS, Go) — secrets resolved at runtime using reference URIs like
  `op://Marketing-Vault/facebook-ads/api-key`. The code never contains actual
  credentials. Fetched, used for the API call, garbage collected from memory.
- **Events API** — 365 days of audit logs. Tracks every secret access (who, what, when).
  Pre-built integrations with Datadog, Splunk, and Panther for anomaly detection.

> **Full details:** see [3-SECURITY.md](3-SECURITY.md) for vault layout, access matrix,
> and breach response playbook.

### Container Structure

The system uses a **Router + per-agent container** architecture:

- **Router** — a tiny stateless service. No brain, no tokens, no secrets. It just
  routes incoming messages to the correct agent container. Tiny attack surface.
- **Agent containers** — each named agent (Jarvis, Nova, Atlas) runs in its own
  isolated container with its own Planner, Policy Engine, Capability Broker, and
  Verifier. Each container holds **only its own** 1Password Service Account token
  scoped to one vault.
- **Worker Manager** — shared across all agents. Spawns/kills ephemeral worker
  containers per task. Workers have zero secrets, zero 1Password access.
- **Agent Manager** — creates, edits, and deletes agent containers. **CRITICAL
  action** — only a human from the Dashboard can trigger it. No agent can create
  another agent.

**Why this split?** If a single Orchestrator holds all agent personas and all
service account tokens, one hack = all vaults compromised. With separate
containers, each agent only has its own scoped token — blast radius is limited
to that one agent's vault.

### The Big Picture

```
┌────────────────────────────────────────────────────────────────────┐
│                          YOU                                       │
│                    (CEO — high-level goals only)                    │
└─────────────────────────┬──────────────────────────────────────────┘
                          │
                  ┌───────▼───────┐
                  │  API SERVER   │  ← CLI / Web Dashboard / Telegram
                  └───────┬───────┘
                          │
                  ┌───────▼───────┐
                  │    ROUTER     │  ← no brain, no tokens
                  │  (msg routing │    just routes messages to the
                  │   only)       │    correct agent container
                  └───────┬───────┘
                          │
        ┌─────────────────┼──────────────────┐
        │                 │                  │
  ┌─────▼─────┐    ┌─────▼─────┐    ┌──────▼─────┐
  │  JARVIS   │    │   NOVA    │    │   ATLAS    │     each agent
  │  CONTAINER│    │ CONTAINER │    │ CONTAINER  │     is its own
  │  SoftDev  │    │ Marketing │    │ Biz Ops    │     isolated process
  └─────┬─────┘    └─────┬─────┘    └──────┬─────┘     with its own
        │                │                  │          1Password token
   (each agent container has the same internal structure below)
        │
        ▼
  ┌──────────────────────────────────────────────────────────┐
  │  INSIDE EACH AGENT CONTAINER                             │
  │  (e.g. Jarvis) — one container per named agent.          │
  │  Each container has its own 1Password Service Account.   │
  │                                                          │
  │  Planner = the agent's brain (the AI / LLM)             │
  │  Policy Engine = a config file with rules (not AI)       │
  │  Broker = a permission system for temp access (not AI)   │
  │                                                          │
  │  ┌──────────┐   ┌──────────┐   ┌──────────┐             │
  │  │ PLANNER  │   │ POLICY   │   │ BROKER   │             │
  │  │          │   │ ENGINE   │   │          │             │
  │  │ THINKS   │   │ ENFORCES │   │ GRANTS   │             │
  │  │ [LLM]    │   │ [rules]  │   │ [leases] │             │
  │  └────┬─────┘   └────┬─────┘   └────┬─────┘             │
  │       └──────────────┼──────────────┘                    │
  │                      │                                   │
  │           ┌──────────▼──────────┐                        │
  │           │     VERIFIER        │                        │
  │           │     CHECKS          │                        │
  │           │   [different LLM]   │                        │
  │           └──────────┬──────────┘                        │
  │                      │                                   │
  │                   Result                                 │
  └──────────────────────────────────────────────────────────┘
                         │
              ┌──────────▼──────────┐
              │   WORKER MANAGER    │  ← shared across all agents
              │   (isolated runner) │    spawns/kills worker containers
              └──────────┬──────────┘
                         │
          ┌──────────────┼──────────────┐
          │              │              │
     ┌────▼────┐   ┌────▼────┐   ┌────▼────┐
     │ Worker  │   │ Worker  │   │ Worker  │
     │ (temp)  │   │ (temp)  │   │ (temp)  │
     │ task-1  │   │ task-2  │   │ task-3  │
     └─────────┘   └─────────┘   └─────────┘
```

### Full Flow Example: Nova Launches a Facebook Ad Campaign

Here's exactly what happens when you send a Telegram message to Nova —
every container, every secret resolution, every security check.

```
1. YOU TYPE ON TELEGRAM
   "Nova, launch the spring sale campaign on Facebook. $200 budget, 7 days."
         │
         ▼
2. TELEGRAM → API SERVER (webhook)
   API Server checks your Telegram user ID → ✅ verified
   Creates a goal in PostgreSQL.
   Publishes to Redis Stream → "goals:incoming"
         │
         ▼
3. ROUTER (always running)
   Reads the goal → message is addressed to Nova.
   Routes to Nova Agent Container.
   (Router never sees secrets — just opens the right door.)
         │
         ▼
4. NOVA AGENT CONTAINER (always running)
   Nova's Planner calls Claude Sonnet → "Break this into tasks..."
   Returns 3 tasks:
     task-1: Create ad creatives (copy + images)      → LOW risk
     task-2: Configure Facebook campaign ($200, 7d)    → MEDIUM risk
     task-3: Launch campaign (spend real money)         → CRITICAL → needs human OK
   Policy Engine approves task-1 and task-2.
   Broker creates capability leases in PostgreSQL.
         │
         ▼
5. WORKER CONTAINER SPAWNS (task-1: ad creatives)
   Worker Manager: docker run --rm worker-image ...
   Worker writes ad copy + image prompts to /workspace/task-1/
   Worker has NO API keys, NO 1Password access.
   Finishes → sends task.complete → container destroyed.
         │
         ▼
6. NOVA REVIEWS task-1
   Planner checks output → "Good. Proceed."
   Verifier gates: ✅ scope check, ✅ no secrets in output
   Verifier AI (GPT-4o): ✅ copy looks professional
         │
         ▼
7. WORKER CONTAINER SPAWNS (task-2: configure campaign)
   Worker: "Set up Facebook campaign with these creatives"
   Worker sends action request → Nova Agent Container handles it:

     Nova uses nova-prod Service Account (scoped to Marketing Vault only)
     Nova calls: client.secrets.resolve("op://Marketing-Vault/facebook-ads/api-key")
            │
            ▼
     1Password Connect (on same VM):
       checks: nova-prod → Marketing Vault → allowed ✅
       returns Facebook API key
            │
            ▼
     Nova calls Facebook Ads API → creates campaign (paused, not launched)
     API key garbage collected from memory (~100ms window)
     Returns to Worker: "Campaign created, ID: 98765, status: PAUSED"

   Worker never saw the Facebook API key.
   Container destroyed.
         │
         ▼
8. TASK-3 ESCALATED (CRITICAL — real money)
   Nova → Router → API Server → Telegram:
   "Campaign ready. $200 budget, 7 days, spring sale.
    Preview: [link]. Ready to launch?"
    [Approve] [Deny] [Edit Budget]
         │
         ▼
9. YOU TAP [Approve] ON TELEGRAM
         │
         ▼
10. NOVA LAUNCHES
    Nova resolves Facebook API key again (same 1Password flow as step 7)
    Calls Facebook API → campaign status: ACTIVE
    API key garbage collected.
         │
         ▼
11. DONE
    Nova → Router → API Server → Telegram:
    "✅ Spring sale campaign live!
     Budget: $200 │ Duration: 7 days │ Ad set ID: 98765
     I'll send daily performance reports."

    Total: 3 tasks │ 2 waves │ 8 minutes │ $0.47 │ you typed 2 messages
```

**Key takeaway:** Nova's container is the **only** thing that ever touches
Marketing Vault. The Router, workers, Jarvis, and Atlas never see those
credentials. If Nova gets hacked, you revoke `nova-prod`, isolate Nova's
container, and every other agent keeps working normally.

---

## The 4 Roles Inside Each Agent

Every named agent (Jarvis, Nova, Atlas) has the same 4-role structure internally.
No single role has full power. They check each other.

### 1. Planner — THINKS (LLM-Powered)

The strategic brain. Receives your goal, breaks it into tasks, picks
technologies, makes decisions, sequences the work.

Planner decides on its own (doesn't ask you):
- Stack choices, auth strategy, task ordering
- Only bothers you for ambiguous goals, budget decisions, or big consequences

### 2. Policy Engine — ENFORCES (Deterministic Rules, NOT an LLM)

A hardcoded rules engine. Cannot be "convinced" or hallucinated around.
If the rule says no, it's no. **An LLM can be tricked. A rules engine can't.**

### 3. Capability Broker — GRANTS (Scoped, Temporary Access)

Issues **leases** — temporary, scoped permissions that expire automatically.
Like a hotel key card that stops working at checkout time.

### 4. Verifier — CHECKS (Deterministic Gates + Different LLM)

Two-phase verification. Deterministic gates first (tests, security scan,
secrets check, scope check). Then AI review from a **different provider**
(e.g., GPT if Planner is Claude).

> **Full details on each role:** see [2-WORKFLOW.md](2-WORKFLOW.md) for how they
> interact in practice, and [3-SECURITY.md](3-SECURITY.md) for risk matrix and guardrails.

---

## Named Agents (Your AI Team)

You can run as many agents as you want. Each one is like a named employee
with their own skills, budget, and track record.

```yaml
agents:
  jarvis:
    name: "Jarvis"
    role: "Software Development"
    planner_model: claude-opus-4-6         # strong reasoning
    worker_model: claude-sonnet-4-5        # cost-effective execution
    verifier_model: openai/gpt-4o          # different provider
    capabilities: [code, test, deploy, github]
    vault: "Dev-Vault"                     # 1Password vault this agent can access
    budget:
      daily: $15
      monthly: $300
      per_task: $5

  nova:
    name: "Nova"
    role: "Marketing"
    planner_model: claude-sonnet-4-5
    worker_model: claude-haiku-4-5
    verifier_model: claude-sonnet-4-5
    capabilities: [facebook-ads, twitter, mailchimp, analytics, web-research]
    vault: "Marketing-Vault"
    budget: { daily: $5, monthly: $100, per_task: $2 }

  atlas:
    name: "Atlas"
    role: "Business Operations"
    planner_model: claude-sonnet-4-5
    worker_model: claude-haiku-4-5
    verifier_model: claude-sonnet-4-5
    capabilities: [stripe, notion, gmail, google-sheets, quickbooks]
    vault: "Ops-Vault"
    budget: { daily: $3, monthly: $60, per_task: $1 }

  sentinel:
    name: "Sentinel"
    role: "DevOps & Infrastructure"
    planner_model: claude-opus-4-6
    worker_model: claude-sonnet-4-5
    verifier_model: openai/gpt-4o
    capabilities: [aws, docker, ssh, monitoring, ssl]
    vault: "Infra-Vault"
    budget: { daily: $10, monthly: $200, per_task: $5 }
```

### Agent Performance Tracking

```
┌──────────┬──────────┬─────────┬────────┬────────┬──────────────┐
│ Agent    │ Tasks    │ Success │ Cost   │ Avg    │ Autonomy     │
│          │ Done     │ Rate    │ Total  │ /Task  │ Score        │
├──────────┼──────────┼─────────┼────────┼────────┼──────────────┤
│ Jarvis   │ 84       │ 91%     │ $189   │ $2.25  │ 94% (great)  │
│ Nova     │ 142      │ 96%     │ $67    │ $0.47  │ 88% (good)   │
│ Atlas    │ 203      │ 99%     │ $41    │ $0.20  │ 97% (great)  │
│ Sentinel │ 31       │ 74%     │ $52    │ $1.68  │ 62% (poor)   │
└──────────┴──────────┴─────────┴────────┴────────┴──────────────┘

Autonomy Score = % of tasks completed without needing human input
```

When an agent underperforms: adjust config, narrow scope, pause, retire, or clone & A/B test.

---

## Ephemeral Workers

Workers are temporary. Spawn per task, destroy after done.

```
Task arrives → Spawn fresh container → Own git branch → Scoped lease → Fresh LLM context
Worker works → Returns structured artifacts → Commits to task branch
Worker finishes → Container destroyed → Branch remains for review → Lease auto-expires
```

No long-running agents. No cross-task contamination. No stale state.

---

## Dashboard

Your control panel. 8 pages. Full spec in [4-DASHBOARD-SPEC.md](4-DASHBOARD-SPEC.md).

| Page | What it does |
|---|---|
| Chat Inbox | Talk to agents in sessions (like ChatGPT). Approve/deny inline |
| Task Board | Kanban view of all tasks. Status, risk, costs, lease TTL |
| Flow View | Step-by-step timeline with [Revert] buttons at every step |
| Config Panel | Edit agent models, budgets, risk matrix, guardrails via UI |
| Review Center | Post-task code diff, full conversation, decision log, cost breakdown |
| Audit Log | Searchable chronological log of every event. Export as CSV/JSON |
| Agent Performance | Monthly scorecards, failure breakdown, trends, actions |
| Agent Management | Create, edit, start, stop, delete agent containers from the UI |

---

## How Revert Works

Every tool execution creates a git commit checkpoint on the task branch.

```
Step 1: write_file     → commit c1
Step 2: npm_install    → commit c2
Step 3: write_file     → commit c3     ← you click [Revert] here
Step 4: git_commit     → commit c4     ← undone
Step 5: merge to main  → commit c5     ← undone
```

Three options:
- **Revert** — roll back to that step, pause task for your decision
- **Revert & Re-run** — roll back, spawn new worker to continue from there
- **Revert Entire Task** — undo everything including merge, delete branch

---

## Infrastructure

### Phase 1: Single VM (Start Here)

```
One VM (24/7) — 4 vCPU, 8GB RAM
├── Orchestrator (Planner + Policy + Broker)
├── Worker Manager
├── Verifier
├── Redis (messages)
├── PostgreSQL (state)
└── Workers (spawned as child processes)

Cost: ~$20-40/month + API usage
```

### Phase 2: Containerized (Production)

```
One VM (24/7) — 8 vCPU, 16GB RAM
├── [Container] Router (message routing only — no brain, no tokens)
├── [Container] Jarvis Agent (Planner + Policy + Broker + Verifier + own 1Password SA)
├── [Container] Nova Agent (Planner + Policy + Broker + Verifier + own 1Password SA)
├── [Container] Atlas Agent (Planner + Policy + Broker + Verifier + own 1Password SA)
├── [Container] Agent Manager (create/edit/delete agents — CRITICAL action, human only)
├── [Container] 1Password Connect API + Sync (private secret REST API)
├── [Container] Worker Manager (rootless Docker, non-root user — shared)
├── [Container] Redis
├── [Container] PostgreSQL
├── [Container] Dashboard (Next.js)
└── Ephemeral worker containers (spawned per task, destroyed after)

Cost: ~$40-80/month + API usage
```

**Why separate agent containers?** If one Orchestrator holds all agent personas
and all 1Password tokens, one hack = all vaults compromised. With separate
containers, each agent has its own scoped token — blast radius is limited.

### Phase 3: Multi-Machine (At Scale)

```
VM 1 (Brain):   Router + Agent Containers + Agent Manager + Redis + PostgreSQL + Dashboard
VM 2 (Workers): Worker Manager + ephemeral containers
VM 3 (GPU):     Image/video/ML workers (if needed)

Cost: $200+/month
```

> **Security details:** see [3-SECURITY.md](3-SECURITY.md)
> **Infrastructure flow (what runs when):** see [5-TECHNICAL-DETAILS.md](5-TECHNICAL-DETAILS.md)

---

## Tech Stack

| Component         | Technology                                |
|-------------------|-------------------------------------------|
| Router            | Lightweight Python/Node service (no LLM, no tokens — routing only) |
| Agent Containers  | Python (FastAPI) or Node.js — one per named agent |
| Agent Manager     | Python (FastAPI) or Node.js — CRITICAL action, human-only |
| Planner LLM       | Claude Opus (strong reasoning)            |
| Worker LLM        | Claude Sonnet / Haiku (cost-effective)    |
| Verifier LLM      | OpenAI GPT-4o (different provider)        |
| Verifier Gates    | jest/pytest + semgrep + gitleaks          |
| Policy Engine     | YAML rules + deterministic evaluator      |
| Capability Broker | Capability classes + lease store (PG)     |
| Workflow State    | PostgreSQL (state machine) + Redis Streams|
| Worker Manager    | Isolated runner API (rootless Docker)     |
| Secrets           | 1Password Connect + Service Accounts + SDK |
| Dashboard         | Next.js + Tailwind + shadcn/ui + WebSocket|
| Chat Interface    | Telegram Bot API (primary), Web UI, CLI   |

---

## What You Need to Start

| Item               | Cost         | Where                   |
|--------------------|--------------|-------------------------|
| Anthropic API key  | Pay per use  | console.anthropic.com   |
| OpenAI API key     | Pay per use  | platform.openai.com     |
| VM Server          | ~$7-48/month | Hetzner / DigitalOcean  |
| Telegram Bot       | Free         | @BotFather on Telegram  |
| GitHub account     | Free         | github.com              |
| Domain (optional)  | ~$10/year    | Cloudflare / Namecheap  |

**Total to start: ~$15-50/month**

> **Step-by-step setup:** see [6-SETUP-GUIDE.md](6-SETUP-GUIDE.md)

---

## Build Order

| Phase | What                                               | When     |
|-------|----------------------------------------------------|----------|
| 1     | Single agent (Jarvis), CLI, basic Planner + Worker | Week 1-2 |
| 2     | Policy Engine + Capability Broker                  | Week 3-4 |
| 3     | Verifier (gates + AI), per-task git branches       | Month 2  |
| 4     | Redis Streams + PostgreSQL state machine           | Month 2  |
| 5     | Telegram bot interface                             | Month 3  |
| 6     | Guardrails, kill switch, overnight mode            | Month 3  |
| 7     | Dockerize workers (ephemeral containers)           | Month 3  |
| 8     | Web dashboard (chat, task board, flow view)        | Month 4  |
| 9     | Router + per-agent containers (split Orchestrator) | Month 4  |
| 10    | Multi-agent support (Nova, Atlas, Sentinel)        | Month 5  |
| 11    | Agent Manager (create/edit/delete agents from UI)  | Month 5  |
| 12    | Performance tracking + agent review                | Month 5  |
| 13    | Revert system                                      | Month 6  |
| 14    | Multi-machine scaling (if needed)                  | Month 6+ |

---

## Inspiration

- **OpenClaw** — Gateway architecture, multi-channel messaging, cron scheduling
- **Temporal.io** — Durable workflow execution, retry semantics
- **Separation of Powers** — AI reasons, policy enforces, broker leases, verifier checks
- **Principle of Least Privilege** — Workers get minimum needed permissions
- **Defense in Depth** — Deterministic gates before AI judgment, different model for verification

**Core innovation: AI controls decisions. Hard policy controls power.
No single component can think AND act AND verify AND grant access.**
