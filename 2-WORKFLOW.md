# AI World — Detailed Workflow

> **Read [1-ARCHITECTURE.md](1-ARCHITECTURE.md) first.** This doc shows the complete step-by-step
> flow of a real task — every message, every decision, every iteration.

Sub-agents are standalone capable, but Master AI validates them for accuracy.

---

## PHASE 1: You → Master AI (Goal Assignment)

```
You (Telegram / Dashboard / CLI):
  "Build me a todo app with user auth. Deploy to staging."
```

That's all you do. One message. Walk away.

---

## PHASE 2: Master AI Thinks (Planning)

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

## PHASE 3: Policy + Lease (Before Any Work Starts)

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

## PHASE 4: Master AI → Sub-Agent (Task 1 — DB Schema)

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

## PHASE 5: Master AI Reviews Task 1 Output

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

## PHASE 6: Master AI → Sub-Agent (Follow-up for Task 1)

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

## PHASE 7: Verifier Checks Task 1

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

## PHASE 8: Master AI → Sub-Agent (Task 2 — REST API)

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

## PHASE 9: Master AI Reviews Task 2 Output

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

## PHASE 10: Task 2 Verification + Tasks 3 & 4

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

## PHASE 11: All Tasks Done — Deploy

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

## The Iteration Pattern (Key Concept)

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
