# AI World — Setup Guide (Step by Step)

## Before You Write Any Code

You need these things first. Think of it like setting up a kitchen before you cook.

---

## Step 1: Get Your API Keys (Free to start, pay as you go)

These are how your AI agents "think." Without these, nothing works.

| What               | Where to get it                        | Cost              |
|--------------------|----------------------------------------|-------------------|
| Anthropic API Key  | https://console.anthropic.com          | Pay per use (~$3-15/M tokens) |
| OpenAI API Key     | https://platform.openai.com            | Pay per use (required for Verifier) |

**Do this now:**
- [ ] Create Anthropic account → generate API key
- [ ] Create OpenAI account → generate API key (needed for Verifier — different provider)
- [ ] Add payment methods to both
- [ ] Save both keys somewhere safe (password manager, NOT in code)

---

## Step 2: Buy a VM (Your 24/7 server)

This is the machine that runs everything. It never sleeps.

### Recommended providers (cheapest to best):

| Provider       | Spec                    | Cost/month | Good for         |
|----------------|-------------------------|------------|------------------|
| Hetzner        | 4 vCPU, 8GB RAM, 80GB  | ~$7-15     | Best value (EU)  |
| DigitalOcean   | 4 vCPU, 8GB RAM, 160GB | ~$48       | Simple UI        |
| Linode/Akamai  | 4 vCPU, 8GB RAM, 160GB | ~$48       | Good docs        |
| AWS EC2        | t3.large, 8GB RAM      | ~$60       | Most flexible    |
| Vultr          | 4 vCPU, 8GB RAM, 160GB | ~$48       | Global locations  |

**For Phase 1 (prototype), start small:**
- Hetzner CX32 or DigitalOcean Basic Droplet
- **OS: Ubuntu 24.04 LTS**
- 4 vCPU, 8GB RAM is enough to start
- You can always resize later

**Do this now:**
- [ ] Pick a provider (Hetzner recommended for cost)
- [ ] Create account
- [ ] Spin up a VM with Ubuntu 24.04 LTS
- [ ] Set up SSH key access (don't use password login)
- [ ] Note down your server IP address

---

## Step 3: Set Up the VM (First SSH into your server)

SSH into your new server and install the basics.

```bash
# Connect to your server
ssh root@YOUR_SERVER_IP

# Update everything
apt update && apt upgrade -y

# Install essentials
apt install -y git curl wget build-essential

# Install Node.js 22 (LTS)
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt install -y nodejs

# Verify
node --version   # should show v22.x
npm --version    # should show 10.x

# Install Docker
curl -fsSL https://get.docker.com | sh

# Verify Docker
docker --version
docker compose version

# Install Redis (for message bus)
docker run -d --name redis --restart always -p 6379:6379 \
  redis:7-alpine redis-server --requirepass yourredispassword

# Install PostgreSQL (for state/memory)
docker run -d --name postgres --restart always \
  -e POSTGRES_PASSWORD=yourpassword \
  -e POSTGRES_DB=aiworld \
  -p 5432:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres:16-alpine
```

**Do this now:**
- [ ] SSH into your server
- [ ] Install Node.js, Docker, Git
- [ ] Start Redis and PostgreSQL containers
- [ ] Test: `docker ps` should show 2 running containers

---

## Step 4: Set Up a Telegram Bot (How you talk to it from your phone)

This is the easiest messaging interface. Free, instant, works on all devices.

### Create the bot:
1. Open Telegram
2. Search for `@BotFather`
3. Send `/newbot`
4. Give it a name: `AI World Bot`
5. Give it a username: `your_aiworld_bot` (must end in `bot`)
6. BotFather gives you a **Bot Token** — save this!

### Get your Telegram User ID:
1. Search for `@userinfobot` on Telegram
2. Send it any message
3. It replies with your user ID (a number like `123456789`)
4. Save this — you'll use it to restrict who can talk to your bot

**Do this now:**
- [ ] Create Telegram bot via BotFather
- [ ] Save the Bot Token
- [ ] Get your Telegram User ID
- [ ] Send a test message to your bot (it won't reply yet, that's OK)

---

## Step 5: Set Up Your Project Structure on the Server

```bash
# Create project directory
mkdir -p /opt/aiworld
cd /opt/aiworld

# Initialize git repo
git init

# Create folder structure
mkdir -p src/orchestrator      # Planner AI, main brain
mkdir -p src/policy-engine     # Rules (YAML config)
mkdir -p src/capability-broker # Permission leases
mkdir -p src/worker-manager    # Spawns/destroys workers
mkdir -p src/verifier          # Reviews output
mkdir -p src/api               # REST API + Telegram bot
mkdir -p src/workers/code      # Code worker template
mkdir -p src/workers/test      # Test worker template
mkdir -p src/workers/research  # Research worker template
mkdir -p config                # Policy rules, guardrails
mkdir -p workspace             # Where workers do their work

# Create config files
touch config/policies.yaml
touch config/guardrails.yaml
touch .env
```

**.env file (put your secrets here, NEVER commit this):**

> **Note:** `.env` with plaintext keys is fine for local prototype development.
> For production / 24/7 autonomy, migrate to Docker secrets or HashiCorp Vault
> (see Security section in ARCHITECTURE.md).

```bash
ANTHROPIC_API_KEY=sk-ant-your-key-here
OPENAI_API_KEY=sk-your-openai-key-here
TELEGRAM_BOT_TOKEN=your-telegram-bot-token
TELEGRAM_ALLOWED_USER_ID=your-telegram-user-id
REDIS_URL=redis://:yourredispassword@localhost:6379
DATABASE_URL=postgresql://postgres:yourpassword@localhost:5432/aiworld
```

**Do this now:**
- [ ] Create the project structure on your server
- [ ] Create the `.env` file with your keys
- [ ] Add `.env` to `.gitignore` immediately: `echo ".env" >> .gitignore`

---

## Step 6: Buy a Domain + Set Up HTTPS (Optional but recommended)

Needed if you want the web dashboard accessible from anywhere.

| What          | Where                    | Cost        |
|---------------|--------------------------|-------------|
| Domain        | Namecheap, Cloudflare    | ~$10/year   |
| SSL/HTTPS     | Let's Encrypt (free)     | Free        |
| Reverse proxy | Caddy or Nginx           | Free        |

**Skip this for Phase 1.** You can access everything via `http://YOUR_IP:PORT` for now.

---

## Complete Shopping List

### Must Have (Day 1)

| Item                | Cost         | Where                     |
|---------------------|--------------|---------------------------|
| Anthropic API Key   | Pay per use  | console.anthropic.com     |
| OpenAI API Key      | Pay per use  | platform.openai.com       |
| VM Server           | ~$7-48/month | Hetzner / DigitalOcean    |
| Telegram Bot        | Free         | @BotFather on Telegram    |

> Both API keys are required. The Verifier uses a **different AI provider**
> than the Planner to eliminate single-model blind spots.

**Total to start: ~$7-48/month + API usage (~$5-30/month depending on use)**

### Nice to Have (Later)

| Item                | Cost         | When                      |
|---------------------|--------------|---------------------------|
| Domain name         | ~$10/year    | When you want web dashboard|
| GitHub repo         | Free         | Version control for your project |

---

## What Order to Build (After Setup)

Once your VM, API keys, and Telegram bot are ready:

```
Week 1-2:  Single agent (Jarvis), CLI, basic Planner + Worker
           (Planner takes goal → breaks into tasks → Worker executes → result)

Week 3-4:  Policy Engine (YAML rules) + Capability Broker (scoped leases)

Month 2:   Verifier (deterministic gates + AI review), per-task git branches
           Redis Streams + PostgreSQL state machine (durable workflow)

Month 3:   Telegram bot interface
           Guardrails, kill switch, overnight mode
           Dockerize workers (ephemeral containers)

Month 4:   Web dashboard (chat inbox, task board, flow view)

Month 5:   Multi-agent support (Nova, Atlas, Sentinel)
           Performance tracking + agent review

Month 6:   Revert system, multi-machine scaling (if needed)
```

> Full 12-phase build order with details: see ARCHITECTURE.md → Build Order.

---

## Quick Validation Test

After Steps 1-5, run this to verify everything works:

```bash
# On your server:

# Test Node.js
node -e "console.log('Node.js works')"

# Test Docker
docker run --rm hello-world

# Test Redis
docker exec redis redis-cli ping
# Should print: PONG

# Test PostgreSQL
docker exec postgres psql -U postgres -d aiworld -c "SELECT 1;"
# Should print: 1

# Test Anthropic API key
curl https://api.anthropic.com/v1/messages \
  -H "x-api-key: $ANTHROPIC_API_KEY" \
  -H "anthropic-version: 2023-06-01" \
  -H "content-type: application/json" \
  -d '{
    "model": "claude-sonnet-4-5-20250929",
    "max_tokens": 100,
    "messages": [{"role": "user", "content": "Say hello"}]
  }'
# Should return a JSON response with Claude saying hello

echo "All systems go!"
```

If all 5 tests pass, you're ready to start coding Phase 1.
