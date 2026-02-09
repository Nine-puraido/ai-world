# AI World

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

### How It Works (30 second version)

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

---

## A Day In Your Life

It's Tuesday morning. You have three AI agents working for you.

### 8:00 AM — You open Telegram / Web Dashboard / CLI

```
Chat: Jarvis (Software Dev)
  Jarvis: Good morning. Overnight I completed:
          ✅ User authentication (JWT + refresh tokens)
          ✅ Database schema (PostgreSQL)
          ✅ API endpoints (12 routes, all tested)

          Blocked on 1 item:
          ⏳ Deploy to production — needs your approval
          [Approve] [Deny]
```

```
Chat: Nova (Marketing)
  Nova: Daily ad report:
        Facebook campaign day 3 of 7.
        Impressions: 12,400 │ Clicks: 348 │ CTR: 2.8%
        Spend so far: $86 of $200 budget.

        Ad B is outperforming Ad A by 3x.
        Recommendation: shift remaining budget to Ad B.
        [Approve] [Deny] [Explain More]
```

```
Chat: Atlas (Business Ops)
  Atlas: Weekly revenue summary:
         Stripe: $4,230 this week (+12% vs last week)
         3 invoices sent, 2 paid, 1 overdue (Client X, 5 days late)

         Should I send a reminder email to Client X?
         [Yes] [No] [Draft first]
```

### 8:02 AM — You make decisions

```
You → Jarvis: Approve deploy
You → Nova: Approve. Shift budget to Ad B.
You → Atlas: Yes, send reminder
```

### 8:03 AM — You put your phone down

The agents handle everything. Jarvis deploys. Nova adjusts the ad campaign.
Atlas sends the email. You check back whenever you want — or don't.

### 6:00 PM — You check the dashboard

You open the web dashboard and see:

```
Today's Summary
───────────────
Jarvis:  6 tasks completed │ $4.20 spent │ 0 failures
Nova:    3 tasks completed │ $1.80 spent │ 0 failures
Atlas:   8 tasks completed │ $0.90 spent │ 0 failures

Total spend: $6.90
All systems normal. No escalations pending.
```

That's it. That's the product.

---

## The Architecture

### Core Principle

**AI controls decisions. Hard policy controls power.**

No single component has full authority. The system has checks and balances —
like a government, not a dictatorship.

```
You (goals) → Planner → Policy → Capability Lease → Worker → Verifier → Result
```

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
        ┌─────────────────┼──────────────────┐
        │                 │                  │
  ┌─────▼─────┐    ┌─────▼─────┐    ┌──────▼─────┐
  │  JARVIS   │    │   NOVA    │    │   ATLAS    │
  │  SoftDev  │    │ Marketing │    │ Biz Ops    │
  └─────┬─────┘    └─────┬─────┘    └──────┬─────┘
        │                │                  │
   (each agent has the same internal structure below)
        │
        ▼
  ┌──────────────────────────────────────────────────────────┐
  │  INSIDE JARVIS — these are not separate systems.         │
  │  They are how Jarvis works internally.                   │
  │                                                          │
  │  Planner = Jarvis's brain (the AI / LLM)                │
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
  │           │   WORKER MANAGER    │                        │
  │           │   (isolated runner) │                        │
  │           └──────────┬──────────┘                        │
  │                      │                                   │
  │       ┌──────────────┼──────────────┐                    │
  │       │              │              │                    │
  │  ┌────▼────┐   ┌────▼────┐   ┌────▼────┐               │
  │  │ Worker  │   │ Worker  │   │ Worker  │               │
  │  │ (temp)  │   │ (temp)  │   │ (temp)  │               │
  │  │ task-1  │   │ task-2  │   │ task-3  │               │
  │  └────┬────┘   └────┬────┘   └────┬────┘               │
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
```

---

## The 4 Roles Inside Each Agent

Every named agent (Jarvis, Nova, Atlas) has the same 4-role structure internally.
No single role has full power. They check each other.

### 1. Planner — THINKS (LLM-Powered)

The strategic brain. Receives your goal, breaks it into tasks, picks
technologies, makes decisions, sequences the work.

**Example: You tell Jarvis "Build me a todo app"**

Planner decides on its own:
- Stack: Next.js + PostgreSQL + Tailwind
- Auth: JWT with refresh tokens
- 5 tasks: DB schema → API → Frontend → Tests → Deploy
- Does NOT ask you "PostgreSQL or MongoDB?" — it just decides

Planner only bothers you when:
- Your goal is ambiguous ("make it better" — better how?)
- The decision involves spending real money beyond budget
- Two options are equally valid with long-term consequences

### 2. Policy Engine — ENFORCES (Deterministic Rules, NOT an LLM)

A hardcoded rules engine. Cannot be "convinced" or hallucinated around.
If the rule says no, it's no. No matter how convincing the argument.

**Why not an LLM?** An LLM can be tricked. A rules engine can't.

```yaml
# Example rules

- action: allow
  scope: write
  path: "/workspace/{task-branch}/**"

- action: block
  scope: write
  path: "/workspace/main/**"
  reason: "Direct writes to main never allowed"

- action: block
  scope: shell
  pattern: "rm -rf"
  reason: "Mass deletion never allowed"

- action: escalate
  scope: deploy
  target: production
  reason: "Production deploys require human approval"

- action: block
  scope: api_call
  when: "daily_spend > budget"
  reason: "Daily budget exceeded"
```

### 3. Capability Broker — GRANTS (Scoped, Temporary Access)

Instead of giving workers permanent access, the broker issues **leases** —
temporary, scoped permissions that expire automatically. Like a hotel key card
that stops working at checkout time.

```json
{
  "leaseId": "lease-abc-123",
  "worker": "worker-task-42",
  "capabilities": ["pkg-install", "build", "test", "git-commit"],
  "scope": {
    "branch": "task-42",
    "workspace": "/workspace/task-42/**"
  },
  "limits": {
    "maxTokens": 100000,
    "maxDurationMinutes": 30,
    "maxCostUsd": 2.00
  },
  "expiresAt": "2026-02-09T10:30:00Z"
}
```

Capabilities are abstract classes (not literal command strings):

```yaml
pkg-install:
  allowed_binaries: [npm, yarn, pip]
  network: [registry.npmjs.org, pypi.org]
  sandbox: no postinstall scripts

build:
  allowed_binaries: [node, npx, tsc, vite]
  network: denied
  sandbox: read source, write dist

test:
  allowed_binaries: [jest, pytest, vitest]
  network: denied
  sandbox: read-only, max 300s
```

### 4. Verifier — CHECKS (Deterministic Gates + Different LLM)

Two-phase verification. The Verifier deliberately uses a **different AI provider**
than the Planner. If the same model writes and reviews, it has the same blind spots.

```
Worker finishes → task branch ready
  │
  ▼
PHASE 1: Deterministic gates (no AI, instant pass/fail)
  ├── Tests pass?              (jest / pytest)
  ├── Security scan clean?     (semgrep)
  ├── No secrets in code?      (gitleaks)
  ├── Build succeeds?          (npm run build)
  └── Diff within scope?       (only task files touched)

  ANY gate fails → REJECT immediately. No AI needed.
  │
  ▼
PHASE 2: AI review (different model — e.g., GPT if Planner is Claude)
  ├── Code quality
  ├── Architecture matches plan
  ├── Edge cases covered
  │
  ├── PASS → enter merge queue → rebase → merge to main
  ├── FAIL (fixable) → send back to worker with feedback
  └── FAIL (suspicious) → escalate to human
```

---

## Named Agents (Your AI Team)

You can run as many agents as you want. Each one is like a named employee
with their own personality, skills, budget, and track record.

### Agent Configuration

```yaml
agents:
  jarvis:
    name: "Jarvis"
    role: "Software Development"
    planner_model: claude-opus-4-6         # strong reasoning
    worker_model: claude-sonnet-4-5        # cost-effective execution
    verifier_model: openai/gpt-4o          # different provider
    capabilities: [code, test, deploy, github]
    budget:
      daily: $15
      monthly: $300
      per_task: $5
    guardrails:
      max_retries: 3
      lease_duration: 30m
      auto_hours: "08:00-22:00"

  nova:
    name: "Nova"
    role: "Marketing"
    planner_model: claude-sonnet-4-5
    worker_model: claude-haiku-4-5
    verifier_model: claude-sonnet-4-5
    capabilities: [facebook-ads, twitter, mailchimp, analytics, web-research]
    budget:
      daily: $5
      monthly: $100
      per_task: $2

  atlas:
    name: "Atlas"
    role: "Business Operations"
    planner_model: claude-sonnet-4-5
    worker_model: claude-haiku-4-5
    verifier_model: claude-sonnet-4-5
    capabilities: [stripe, notion, gmail, google-sheets, quickbooks]
    budget:
      daily: $3
      monthly: $60
      per_task: $1

  sentinel:
    name: "Sentinel"
    role: "DevOps & Infrastructure"
    planner_model: claude-opus-4-6
    worker_model: claude-sonnet-4-5
    verifier_model: openai/gpt-4o
    capabilities: [aws, docker, ssh, monitoring, ssl]
    budget:
      daily: $10
      monthly: $200
      per_task: $5
    guardrails:
      lease_duration: 60m   # infra tasks take longer
```

### Talking to Your Agents

On Telegram, each agent gets its own chat — or you tag them in one chat:

```
You:       @jarvis build a landing page for the todo app
Jarvis:    On it. 4 tasks planned. All low-risk. Executing.

You:       @nova run Facebook ads for it, $200 budget, 7 days
Nova:      Campaign plan ready.
           Ad A: feature-focused
           Ad B: testimonial-focused
           Target: developers, 25-40
           [Approve] [Deny]

You:       Approve
Nova:      Campaign live. I'll report daily at 9am.

You:       @atlas how many signups today?
Atlas:     47 signups. 12 from Twitter, 28 from ads, 7 organic.
```

### Agent Performance Tracking

Every agent's work is tracked automatically:

```
Per task:   success/fail, cost, duration, retries, human escalations
Per day:    tasks completed, total cost, failure rate
Per month:  success rate trend, avg cost/task, autonomy score
```

**Monthly review:**

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

**When an agent underperforms, you can:**
- **Adjust config** — upgrade model, increase limits, refine prompt
- **Narrow scope** — remove capabilities it's bad at
- **Pause** — stop new tasks while you investigate
- **Retire** — shut down and redistribute work
- **Clone & A/B test** — duplicate config, tweak, compare

---

## Detailed Workflow: "Build a Todo App"

This is the complete flow — every message, every decision, every iteration.
Sub-agents are standalone capable, but Master AI validates them for accuracy.

---

### PHASE 1: You → Master AI (Goal Assignment)

```
You (Telegram / Dashboard / CLI):
  "Build me a todo app with user auth. Deploy to staging."
```

That's all you do. One message. Walk away.

---

### PHASE 2: Master AI Thinks (Planning)

Master AI (Planner) receives your goal and reasons through it alone.
It does NOT ask you any questions — it has the skills to decide.

```
MASTER AI (internal reasoning):
│
│  Goal: "Todo app with auth, deploy to staging"
│
│  Thinking...
│  - This needs a frontend, backend API, database, auth system, and deployment.
│  - Stack decision: Next.js (React + SSR), PostgreSQL (relational data),
│    Tailwind (styling). I'll use JWT for auth with refresh tokens.
│  - I'll break this into sequential tasks. DB first because API depends on
│    it. Tests can run in parallel. Deploy is last and needs human OK.
│
│  Creates task plan:
│
│  TASK 1: "Design the PostgreSQL database schema for a todo app
│           with users table and todos table. Include migrations."
│           → assign to: code worker
│           → risk: LOW
│           → depends on: nothing
│
│  TASK 2: "Build a REST API with Express.js. Endpoints: CRUD for todos,
│           auth (register/login/refresh). Use JWT. Connect to PostgreSQL.
│           Use Zod for input validation."
│           → assign to: code worker
│           → risk: LOW
│           → depends on: TASK 1 (needs DB schema)
│
│  TASK 3: "Build the React frontend with Next.js + Tailwind + shadcn/ui.
│           Pages: login, register, todo list, create/edit todo.
│           Connect to the REST API."
│           → assign to: code worker
│           → risk: MEDIUM (needs npm packages from internet)
│           → depends on: TASK 2 (needs API to exist)
│
│  TASK 4: "Write tests for the API and frontend. Target 85%+ coverage."
│           → assign to: code worker
│           → risk: LOW
│           → depends on: TASK 2 + TASK 3
│
│  TASK 5: "Deploy to staging (Vercel for frontend, Railway for API)"
│           → assign to: devops worker
│           → risk: HIGH (deployment = needs approval)
│           → depends on: ALL tasks
```

---

### PHASE 3: Policy + Lease (Before Any Work Starts)

Each task goes through Policy Engine and Capability Broker before execution.

```
POLICY ENGINE checks each task:
  TASK 1: actions=[write_to_branch] → LOW → ALLOW
  TASK 2: actions=[write, pkg_install] → MEDIUM → ALLOW (work hours)
  TASK 3: actions=[write, pkg_install] → MEDIUM → ALLOW (work hours)
  TASK 4: actions=[run_tests] → LOW → ALLOW
  TASK 5: actions=[deploy_staging] → HIGH → NEEDS PLANNER APPROVAL

CAPABILITY BROKER issues leases:
  TASK 1: caps=[build, git-commit],        branch=task-1, 30min
  TASK 2: caps=[pkg-install, build, test], branch=task-2, 45min
  TASK 3: caps=[pkg-install, build],       branch=task-3, 45min
  TASK 4: caps=[test],                     branch=task-4, 30min
  TASK 5: WAITING (queued until tasks 1-4 done + Planner approves)
```

---

### PHASE 4: Master AI → Sub-Agent (Task 1 — DB Schema)

Master sends a detailed prompt to a fresh worker.

```
MASTER AI → WORKER (task-1):

  "Design the PostgreSQL database schema for a todo app.

   Requirements:
   - users table: id, email, password_hash, created_at
   - todos table: id, user_id (FK), title, description, completed, created_at
   - Include migration files using db-migrate
   - Add seed data for testing

   Your branch: task-1
   Your workspace: /workspace/task-1/
   Capabilities: [build, git-commit]
   Time limit: 30 minutes"
```

Worker works autonomously — writes files, creates schema, writes migrations.

```
WORKER (task-1) works...
│
├── Creates src/db/schema.sql
├── Creates src/db/migrations/001-create-users.sql
├── Creates src/db/migrations/002-create-todos.sql
├── Creates src/db/seed.sql
├── Commits to branch task-1
│
└── WORKER → MASTER AI:
    "Done. Created database schema with users and todos tables.
     2 migration files + seed data. Committed to task-1."

    Artifacts:
    {
      files_changed: 4,
      lines_added: 89,
      tests: null (no tests for schema),
      cost: $0.32,
      duration: "2m 14s"
    }
```

No questions needed. Worker handled it alone. Master validates.

---

### PHASE 5: Master AI Reviews Task 1 Output

Master AI doesn't just trust the worker blindly. It reviews.

```
MASTER AI (reviewing task-1 output):
│
│  Thinking...
│  - Schema has users and todos tables ✅
│  - Foreign key on todos.user_id → users.id ✅
│  - Migrations are sequential ✅
│  - Missing: I should have asked for an index on todos.user_id
│    for query performance. Let me add a follow-up.
│
│  Decision: Output is 90% good. Need one small fix.
```

---

### PHASE 6: Master AI → Sub-Agent (Follow-up for Task 1)

Master sends a follow-up prompt to a NEW worker (ephemeral — old one is gone).

```
MASTER AI → WORKER (task-1-followup):

  "On branch task-1, add a database index on todos.user_id
   for query performance. Create a new migration file:
   003-add-index-todos-user-id.sql"
```

```
WORKER (task-1-followup) works...
│
├── Creates src/db/migrations/003-add-index-todos-user-id.sql
├── Commits to branch task-1
│
└── WORKER → MASTER AI:
    "Done. Added index migration."
    { files_changed: 1, lines_added: 3, cost: $0.08 }
```

```
MASTER AI (reviewing):
  ✅ Good. Task 1 is complete. Send to Verifier.
```

**This is the iteration loop:**
```
Master assigns prompt → Worker executes → Master reviews output
  → If good: move to Verifier
  → If needs improvement: send follow-up prompt → Worker executes → Master reviews again
  → Repeat until Master is satisfied
```

---

### PHASE 7: Verifier Checks Task 1

```
VERIFIER (task-1):

  DETERMINISTIC GATES:
    ✅ Build: migrations parse correctly
    ✅ SAST: no SQL injection patterns
    ✅ Secrets: no credentials in files
    ✅ Scope: only task-1 branch files modified

  AI REVIEW (GPT-4o):
    ✅ Schema is normalized, relationships correct
    ✅ Index on FK is good practice

  RESULT: PASS → enters merge queue → merged to main
```

---

### PHASE 8: Master AI → Sub-Agent (Task 2 — REST API)

Now Master assigns Task 2. This one is more complex — the worker will
have questions.

```
MASTER AI → WORKER (task-2):

  "Build a REST API with Express.js for the todo app.

   The database schema is already done (merged to main). Pull latest main.

   Requirements:
   - Endpoints: POST /auth/register, POST /auth/login, POST /auth/refresh
   - Endpoints: GET /todos, POST /todos, PUT /todos/:id, DELETE /todos/:id
   - JWT auth with refresh tokens (access: 15min, refresh: 7 days)
   - Zod validation on all inputs
   - Error handling middleware
   - Connect to PostgreSQL using the schema from task-1

   Branch: task-2
   Capabilities: [pkg-install, build, test, git-commit]
   Time limit: 45 minutes"
```

Worker starts working but hits a decision point:

```
WORKER (task-2) works...
│
├── Pulls latest main (has task-1 schema)
├── Creates package.json, installs express, pg, zod, jsonwebtoken
│   (permission auto-granted by Policy — pkg-install capability)
│
├── Creates src/index.js (Express server setup)
├── Creates src/routes/auth.js (register, login, refresh)
├── Creates src/routes/todos.js (CRUD)
│
├──── WORKER HAS A QUESTION ────
│
│  WORKER → MASTER AI:
│    "The task says JWT auth, but should I store refresh tokens
│     in the database or use httpOnly cookies? DB storage is more
│     secure for revocation but adds a DB table. Cookies are simpler
│     but harder to revoke."
│
│     Options:
│     A) Store in database (more secure, adds refresh_tokens table)
│     B) httpOnly cookies (simpler, no extra table)
│
│  MASTER AI (Planner thinks):
│    "Database storage. We need the ability to revoke tokens
│     when users change passwords or log out. Add a refresh_tokens
│     table: id, user_id, token_hash, expires_at, revoked_at."
│
│  MASTER AI → WORKER:
│    "Use database storage. Add a refresh_tokens table:
│     id, user_id, token_hash, expires_at, revoked_at.
│     This allows token revocation on logout and password change."
│
├──── WORKER CONTINUES WITH MASTER'S DECISION ────
│
├── Creates src/db/migrations/004-create-refresh-tokens.sql
├── Creates src/middleware/auth.js (JWT verification middleware)
├── Creates src/middleware/validate.js (Zod validation middleware)
├── Creates src/middleware/errorHandler.js
│
├──── WORKER HAS ANOTHER QUESTION ────
│
│  WORKER → MASTER AI:
│    "Should I add rate limiting to the auth endpoints?
│     It's not in the requirements but it's a security best practice."
│
│  MASTER AI (Planner thinks):
│    "Good thinking. Yes, add it. 10 attempts per minute per IP
│     on /auth/login and /auth/register."
│
│  MASTER AI → WORKER:
│    "Yes, add rate limiting. Use express-rate-limit.
│     10 req/min per IP on auth endpoints."
│
├──── WORKER NEEDS A PERMISSION ────
│
│  WORKER → MASTER AI:
│    Permission request: pkg-install for "express-rate-limit"
│
│  POLICY ENGINE: ALLOW (pkg-install capability, npm registry)
│
│  MASTER AI → WORKER:
│    Permission granted.
│
├── Installs express-rate-limit
├── Adds rate limiting to auth routes
├── Writes tests for all endpoints (12 tests)
├── Runs tests → 12 passed, 0 failed
├── Commits to branch task-2
│
└── WORKER → MASTER AI:
    "Done. REST API with auth, CRUD, validation, rate limiting, tests."

    Artifacts:
    {
      files_changed: 9,
      lines_added: 342,
      tests: { passed: 12, failed: 0, coverage: 87 },
      cost: $0.82,
      duration: "8m 14s",
      questions_asked: 2,
      permissions_requested: 2
    }
```

---

### PHASE 9: Master AI Reviews Task 2 Output

```
MASTER AI (reviewing task-2 output):
│
│  Thinking...
│  - All endpoints implemented ✅
│  - JWT with DB-stored refresh tokens (as I instructed) ✅
│  - Zod validation on all inputs ✅
│  - Rate limiting on auth ✅
│  - 12 tests, 87% coverage ✅
│  - Error handling middleware ✅
│
│  Hmm, but I notice there's no CORS setup. The frontend
│  (task-3) will need to call this API from a different origin.
│  Let me send a follow-up.
│
│  Decision: 95% good. One missing piece.
```

```
MASTER AI → WORKER (task-2-followup):

  "On branch task-2, add CORS middleware.
   Allow origin: http://localhost:3000 (dev) and
   https://todo.yourdomain.com (prod).
   Use the cors npm package."
```

```
WORKER (task-2-followup):
│
├── Installs cors
├── Adds CORS config to src/index.js
├── Commits
│
└── "Done. CORS added for dev and prod origins."
```

```
MASTER AI: ✅ Task 2 complete. Send to Verifier.
```

---

### PHASE 10: Task 2 Verification + Tasks 3 & 4

Verifier checks task-2 → PASS → merged to main.

Now Master starts tasks 3 and 4. Task 3 depends on task-2 (needs the API).
Task 4 needs both task-2 and task-3.

```
MASTER AI → WORKER (task-3):
  "Build the React frontend with Next.js + Tailwind + shadcn/ui.
   The API is at /api (proxy in next.config.js).
   Pages: login, register, todo list with create/edit/delete.
   Use the API endpoints from task-2 (already merged to main)."
```

Task 3 worker runs. It asks:

```
WORKER (task-3) → MASTER AI:
  "Should I add a dark mode toggle? Not in requirements
   but shadcn/ui supports it easily."

MASTER AI → WORKER:
  "No. Keep scope minimal. Ship first, add features later."
```

Worker finishes. Master reviews:

```
MASTER AI (reviewing task-3):
│
│  Frontend looks good. All pages work. But... the loading
│  states are missing on async operations. When you click
│  "Create Todo" there's no spinner or disabled state.
│  That's a bad UX.
│
│  Decision: Send follow-up.
```

```
MASTER AI → WORKER (task-3-followup):
  "Add loading states to all async operations:
   - Login/register buttons: show spinner, disable on submit
   - Todo create/edit: show spinner
   - Todo list: show skeleton loader on initial fetch
   - Delete: show confirmation dialog then spinner"
```

Worker fixes → Master reviews → ✅ satisfied → Verifier → PASS → merged.

Task 4 (tests) runs similarly — worker writes tests, Master reviews
coverage, sends follow-up if gaps found, iterates until satisfied.

---

### PHASE 11: All Tasks Done — Deploy

Tasks 1-4 are merged. Master AI moves to task 5 (deploy).

```
MASTER AI (internal):
│
│  All 4 code tasks are merged and verified. Time for deploy.
│  Deploy is HIGH risk — checking policy...
│  Policy says: HIGH → needs Planner approval.
│  As Planner, I approve this — all code is tested and verified.
│
│  But wait — this is a deploy action. Let me also check with
│  the human since it's their infrastructure.
│
│  Decision: Ask human for final OK before deploying.
```

```
MASTER AI → You (Telegram / Dashboard / CLI):

  "All code tasks complete and merged.
   - 4 tasks merged to main
   - 24 tests passing, 87% coverage
   - Code reviewed by Verifier (GPT-4o)
   - No security issues found

   Ready to deploy to staging.
   Frontend → Vercel
   API → Railway

   [Approve] [Deny] [Show me the code first]"
```

```
You: Approve
```

```
MASTER AI → WORKER (task-5):
  "Deploy the todo app to staging.
   Frontend: vercel --prod (use VERCEL_TOKEN)
   API: railway up (use RAILWAY_TOKEN)
   After deploy, verify both are responding with a health check."
```

```
WORKER (task-5):
│
├── Deploys frontend to Vercel → https://todo-staging.vercel.app
├── Deploys API to Railway → https://todo-api-staging.railway.app
├── Health check: GET / → 200 OK ✅
├── Health check: GET /api/health → 200 OK ✅
│
└── "Deployed. Both services healthy."
```

```
MASTER AI → You (Telegram / Dashboard / CLI):

  "✅ Deployed!

   Frontend: https://todo-staging.vercel.app
   API: https://todo-api-staging.railway.app

   Both health checks passing.

   Summary:
   - 5 tasks completed
   - 2 iterations (added index + added CORS)
   - 2 worker questions answered
   - Total cost: $2.84
   - Total time: 32 minutes
   - 24 tests, 87% coverage

   Want me to do anything else with this project?"
```

---

### The Iteration Pattern (Key Concept)

This is the core loop that repeats for EVERY task:

```
┌──────────────────────────────────────────────────────────────┐
│                                                              │
│  MASTER AI writes a detailed prompt                          │
│       │                                                      │
│       ▼                                                      │
│  WORKER executes (writes code, calls APIs, whatever)         │
│       │                                                      │
│       ├── Worker handles trivial decisions itself             │
│       │   (variable names, local code structure)              │
│       │                                                      │
│       ├── Worker asks Master for big decisions                │
│       │   (architecture, libraries, scope changes)            │
│       │   Master answers → Worker continues                   │
│       │                                                      │
│       ├── Worker asks Master for permissions                  │
│       │   Policy checks → grant/deny → Worker continues      │
│       │                                                      │
│       ▼                                                      │
│  WORKER returns result + artifacts                            │
│       │                                                      │
│       ▼                                                      │
│  MASTER AI reviews the output                                │
│       │                                                      │
│       ├── Good enough? ──YES──► Send to Verifier             │
│       │                                                      │
│       └── Needs work? ──YES──► Write follow-up prompt         │
│                                 (spawn new worker, same branch)│
│                                 Loop back to top ↑            │
│                                                              │
│  After Verifier passes:                                      │
│  Master gives the NEXT task prompt to a new worker            │
│  Loop continues until all tasks are done                      │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

**Why this works:**

Sub-agents (workers) are fully capable — they can write an entire API
on their own. But by routing everything through Master AI, you get:

- **Better decisions** — Master has the full project context, workers only
  see their single task. Master catches things workers miss (like CORS).
- **Consistency** — Master ensures all tasks follow the same architecture
  decisions (e.g., "we use Zod everywhere, not express-validator").
- **Quality control** — Master reviews output and sends follow-up prompts
  to fix issues before they reach the Verifier.
- **Coordination** — Master knows task-3 depends on task-2's API format,
  so it includes the right details in task-3's prompt.

Think of it like a senior engineer reviewing a junior's PR —
the junior can do the work, but the review catches mistakes
and ensures alignment with the bigger picture.

---

## How Agents Talk to Workers (Communication Protocol)

Every message uses a strict envelope:

```json
{
  "id": "msg-001",
  "taskId": "task-42",
  "type": "permission.request",
  "replyTo": null,
  "ts": "2026-02-09T10:02:45Z",
  "idempotencyKey": "idem-task-42-perm-001",
  "payload": { ... }
}
```

- `id` — unique message ID
- `replyTo` — links replies to original messages (correlation)
- `idempotencyKey` — prevents duplicate processing on retry
- Transport: **Redis Streams** (durable, survives crashes)
- State: **PostgreSQL** (task state machine, lease tracking, audit log)

Workers send structured artifacts, not raw text:

```json
{
  "artifactType": "test_result",
  "data": {
    "passed": 12,
    "failed": 0,
    "coverage": 87.3,
    "durationMs": 4230,
    "exitCode": 0
  }
}
```

Workers send heartbeats every 10 seconds. If no heartbeat for 30 seconds,
the lease is revoked and the container is killed.

---

## Decision Authority

```
Question                               Who decides?
─────────────────────────────────────  ──────────────────────────
"React or Vue?"                        Planner (tech choice)
"This button blue or green?"           Worker (trivial, local)
"Need to install npm packages"         Policy auto-approves (LOW)
"Can I write to main branch?"          Policy BLOCKS (rule)
"Can I run rm -rf?"                    Policy BLOCKS (rule)
"Deploy to staging?"                   Planner approves (HIGH)
"Deploy to production?"                Policy ESCALATES → human
"Tests failing, should I refactor?"    Planner (re-planning)
"Budget limit hit"                     Kill switch → pause + notify
"It's 3am, medium-risk task"           Queue for morning (time window)
```

The order of authority:
1. **Capability check** — does the lease even allow this? (deterministic)
2. **Policy check** — do the rules allow this? (deterministic)
3. **Planner recommendation** — is this a good idea? (LLM)
4. **Policy validation** — does the LLM's recommendation violate any rules? (deterministic)

**Policy always has final say.** If the Planner says "yes" but Policy says "no",
the answer is no. An LLM can be tricked. Rules can't.

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
    daily_dollar_limit: $10.00
    per_task_token_limit: 100000
    per_task_dollar_limit: $2.00

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

## Ephemeral Workers

Workers are temporary. Spawn per task, destroy after done.

```
Task arrives
  → Spawn fresh container
  → Own git branch (task-42)
  → Scoped capability lease (30 min)
  → Fresh LLM context

Worker does the work
  → Returns structured artifacts
  → Commits to task branch

Worker finishes
  → Container destroyed
  → Branch remains for review
  → Lease auto-expires
```

No long-running agents. No cross-task contamination. No stale state.
If a worker hangs, kill it — no side effects.

---

## Dashboard

Your control panel for the entire system.

### Chat Inbox
Talk to each agent in separate sessions (like ChatGPT). Each session has
its own context. Approve/deny actions inline. Fork sessions to continue
with modifications.

### Task Board
Kanban view of all tasks across all agents. See status, risk level,
lease TTL, costs. Approve or deny pending actions.

### Flow View
Step-by-step timeline of every action in a task. Every tool call,
every permission request, every decision. Each step has a **[Revert]**
button — click it to roll back to that point.

### Config Panel
Edit all settings through the UI:
- Agent models, budgets, capabilities
- Risk matrix levels
- Guardrails and time windows
- Notification preferences

### Review Center
Post-task deep review:
- Full code diff
- Complete agent ↔ worker conversation
- Every decision and who made it (worker, planner, policy)
- Cost breakdown per LLM provider
- [Approve & Keep] or [Revert Entire Task] or [Re-run]

### Audit Log
Searchable chronological log of every event in the system. Filter by
agent, task, time, event type. Export as CSV/JSON.

### Agent Performance
Monthly scorecards per agent:
- Tasks completed, success rate, failure patterns
- Total cost, average cost per task
- Autonomy score (% of tasks without human input)
- Trend: getting better or worse?

---

## Telegram Bot Message Workflow

This belongs to the interface layer (not infrastructure).
Infrastructure hosts services; this section explains message flow.

```
You (Telegram)
  │ send message / tap button
  ▼
Telegram API
  │ webhook update
  ▼
API Server (/webhooks/telegram)
  │ validate secret token + user/chat allowlist + dedupe update_id
  │
  ├── inbound event -> Redis Stream (events.inbox)
  └── persist session/task metadata -> PostgreSQL
                  │
                  ▼
           Orchestrator (Planner/Policy/Broker)
                  │
                  ├── progress/status events -> Redis Stream (events.notifications)
                  └── approval requests -> pending inbox state
                                   │
                                   ▼
                         Telegram Notifier Service
                                   │ sendMessage/editMessageText
                                   ▼
                              Telegram Chat
```

Approval round-trip:
- User taps `[Approve]` / `[Deny]` (Telegram callback query)
- API Server verifies callback origin, chat identity, and decision idempotency
- Decision event is written (`approval.decision`) and workflow resumes
- Policy/Broker re-checks permissions, then task continues or is cancelled
- Notifier edits the original Telegram message with final status

Reliability rules:
- Inbound dedupe by Telegram `update_id` + internal `idempotencyKey`
- API replies `200` quickly; heavy work is async through streams
- Retry with backoff for Telegram `429/5xx`; dead-letter queue on repeated failure
- Chat delivery failure never blocks task execution

Security rules:
- Bot token stored in secrets manager (`/run/secrets` or Vault), never in worker containers
- Only API/Notifier can call Telegram APIs; workers have no Telegram credentials
- Per-user RBAC: only approved users can issue goals or approve actions
- Command allowlist + rate limits on inbound chat commands

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

Everything runs as processes on one machine. Prove the concept.

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

Everything in containers, workers in isolated ephemeral containers.

```
One VM (24/7) — 8 vCPU, 16GB RAM
├── [Container] Orchestrator
├── [Container] Worker Manager (rootless Docker, non-root user)
├── [Container] Redis
├── [Container] PostgreSQL
├── [Container] Dashboard (Next.js)
└── Ephemeral worker containers (spawned per task, destroyed after)

Cost: ~$40-80/month + API usage
```

Security:
- Workers run with **no-new-privileges**, dropped capabilities, read-only rootfs
- Orchestrator has **no Docker access** (can't spawn containers directly)
- Workers **never receive API keys** — all external calls go through a tool proxy
  on the Orchestrator (Orchestrator holds the real keys, workers just request actions)
- Signed message envelopes — every Redis message includes a signature,
  so a compromised worker can't fake `permission.granted` or `task.done`
- Network egress default-deny — workers can only reach whitelisted domains
  (npm registry, pypi) through a proxy. Internal IPs and cloud metadata
  (`169.254.169.254`) are blocked
- `npm install` runs with `--ignore-scripts` (no postinstall execution).
  Lockfile required. No arbitrary package installs without Policy approval

### Phase 3: Multi-Machine (At Scale)

```
VM 1 (Brain):   Orchestrator + Redis + PostgreSQL + Dashboard
VM 2 (Workers): Worker Manager + ephemeral containers
VM 3 (GPU):     Image/video/ML workers (if needed)

Cost: $200+/month
```

---

## Security

### Core Rule

**Containers alone do not stop prompt injection.**
Prompt injection happens at the LLM level — the LLM gets tricked into making
bad tool calls. The container doesn't help because the tool calls look "legitimate"
from the container's perspective.

Prompt injection is stopped by:
tool-level policy checks + signed identity + network egress control + treating
all external data as untrusted input (never as instructions).

### What's Built In (Day 1)

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

### Known Concerns (Needs Future Discussion)

These are real risks that are acceptable during prototype/development but
must be resolved before running 24/7 autonomous production workloads.

**1. Worker-Manager blast radius**

The Worker Manager can spawn/kill any worker container and access all mounted
workspaces. If compromised, it controls everything.

- *Current mitigation:* rootless Docker, non-root user, no-new-privileges
- *Concern:* rootless Docker is still powerful enough to mount filesystems
  and control all worker containers
- *Proposed:* evaluate **gVisor** or **Firecracker** micro-VMs for stronger
  isolation. These trap compromised processes in a much tighter sandbox than
  Docker. Trade-off: more complex setup, higher resource overhead

**2. Service identity (mTLS)**

Currently, services authenticate via signed message envelopes (HMAC).
This is good for prototype but has limits — shared HMAC keys can leak.

- *Concern:* if a worker extracts the signing key, it can impersonate
  any service
- *Proposed:* upgrade to **mTLS** (mutual TLS) between orchestrator,
  worker-manager, and Redis gateway. Each service gets its own certificate.
  Per-service RBAC so workers literally cannot publish on orchestrator channels.
  Trade-off: certificate management complexity

**3. Secret rotation**

API keys and tokens are currently static (set once in Docker secrets/Vault).

- *Concern:* if a key leaks, it stays valid until manually rotated
- *Proposed:* automatic rotation on a schedule. Short-lived scoped tokens
  for workers (expire in minutes, not days). Secret redaction in all logs
  (scan logs for key patterns, replace with `[REDACTED]`).
  Trade-off: rotation logic adds complexity

**4. Supply chain attacks (npm/pip packages)**

Workers can install packages from public registries. Malicious packages exist.

- *Current mitigation:* `--ignore-scripts`, lockfile required, Policy approval
- *Concern:* a dependency of a dependency could still contain malicious code
  that runs at import time (not just postinstall)
- *Proposed:* private registry proxy (Verdaccio/Artifactory) that only allows
  pre-approved packages. Or: pre-baked worker images with common packages
  already installed, no runtime installs at all.
  Trade-off: maintenance burden of approving packages

**5. Prompt injection via legitimate tools**

Blocking `shell()` helps, but `write_file()` + `git_commit()` is still
powerful. A prompt-injected worker could write malicious code and commit it.

- *Current mitigation:* Verifier security scan (semgrep) + scope check +
  different LLM review
- *Concern:* sophisticated injections could write code that passes static
  analysis but behaves maliciously at runtime
- *Proposed:* taint tracking — mark all external inputs (web content, file
  reads, API responses) as "untrusted data" that cannot directly trigger
  privileged tool calls. Policy must re-validate any action influenced by
  external input. Trade-off: complex to implement, may slow down workers

**6. Immutable audit log**

Current audit log is in PostgreSQL. If the database is compromised,
logs can be altered to hide an attack.

- *Current mitigation:* PostgreSQL with append-only application logic
- *Proposed:* write-once storage (S3 with Object Lock, or a dedicated
  append-only log service). Anomaly detection alerts on unusual patterns
  (e.g., worker making 100x more tool calls than normal).
  Trade-off: additional infrastructure cost

### Security Roadmap

| Phase | Security Level | What's in place |
|---|---|---|
| Prototype (Phase 1-3) | Dev-safe | No shell, file isolation, policy checks, tool proxy, signed messages |
| Supervised autonomy (Phase 4-7) | Work-hours safe | + network egress control, lockfile enforcement, scope checks, Verifier gates |
| 24/7 autonomy (Phase 8+) | Production-safe | + mTLS, secret rotation, gVisor/Firecracker, private registry, taint tracking, immutable logs |

> **Bottom line:** the prototype is safe for development with you watching.
> 24/7 unsupervised autonomy requires the upgrades in the roadmap above.
> We'll revisit each "Proposed" item when we reach that phase.

---

## Tech Stack

| Component         | Technology                                |
|-------------------|-------------------------------------------|
| Orchestrator      | Python (FastAPI) or Node.js (Express)     |
| Planner LLM       | Claude Opus (strong reasoning)            |
| Worker LLM        | Claude Sonnet / Haiku (cost-effective)    |
| Verifier LLM      | OpenAI GPT-4o (different provider)        |
| Verifier Gates    | jest/pytest + semgrep + gitleaks          |
| Policy Engine     | YAML rules + deterministic evaluator      |
| Risk Matrix       | Single YAML file (source of truth)        |
| Capability Broker | Capability classes + lease store (PG)     |
| Workflow State    | PostgreSQL (state machine) + Redis Streams|
| Worker Manager    | Isolated runner API (rootless Docker)     |
| Merge Queue       | Custom rebase queue                       |
| Secrets           | Docker secrets / HashiCorp Vault          |
| Dashboard         | Next.js + Tailwind + shadcn/ui + WebSocket|
| Chat Interface    | Telegram Bot API (primary), Web UI        |

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
| 9     | Multi-agent support (Nova, Atlas, Sentinel)        | Month 5  |
| 10    | Performance tracking + agent review                | Month 5  |
| 11    | Revert system                                      | Month 6  |
| 12    | Multi-machine scaling (if needed)                  | Month 6+ |

---

## Inspiration

- **OpenClaw** — Gateway architecture, multi-channel messaging, cron scheduling
- **Temporal.io** — Durable workflow execution, retry semantics
- **Separation of Powers** — AI reasons, policy enforces, broker leases, verifier checks
- **Principle of Least Privilege** — Workers get minimum needed permissions
- **Defense in Depth** — Deterministic gates before AI judgment, different model for verification

**Core innovation: AI controls decisions. Hard policy controls power.
No single component can think AND act AND verify AND grant access.**
