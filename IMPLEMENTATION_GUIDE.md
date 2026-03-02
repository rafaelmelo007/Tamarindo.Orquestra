# Orquestra — Complete Project & Learning Guide

## Project Overview

An AI workforce management platform where you create, manage, and orchestrate autonomous AI agents as employees of a virtual company. Each agent has its own role, AI model, system prompt, tools, and budget. Agents execute scheduled tasks autonomously, participate in simulated daily scrums, share status with each other, access company artifacts, and track token costs — all while you review outcomes as the CEO.

**Core concept:** Your AI company runs itself. You define the team, assign the work, and review the results.

**Repo:** `github.com/yourusername/orquestra`

---

## Learning Path & Resources

### Bun Runtime + TypeScript Backend

**"Bun - The Complete Guide To Fullstack Development With TS/JS" (Udemy)**
Practical, API-first approach. Covers Bun's built-in server, test runner, bundler, package manager, and database integrations.
→ https://www.udemy.com/course/bun-the-guide-to-full-stack-dev-with-typescript-and-javascript/

**"Mastering Bun - The Modern Fullstack Development" (Udemy)**
Deep dive into Bun APIs, HTTP server management, file operations. Covers transition from Node.js.
→ https://www.udemy.com/course/mastering-bun-the-modern-fullstack-development/

**Official Docs:** https://bun.sh/docs

### TypeScript

**"Understanding TypeScript - 2025 Edition" by Maximilian Schwarzmüller (Udemy)**
200,000+ students. Covers basics through advanced (generics, decorators, mapped types). Includes React + Node integration.
→ https://www.udemy.com/course/understanding-typescript/

**"TypeScript Masterclass 2025 Edition - React + NodeJS Project" (Udemy)**
Full-stack project with React + ShadcnUI + TanStack Query on frontend, Node + Express + MongoDB on backend. All TypeScript.
→ https://www.udemy.com/course/typescript-course/

### React

**"Complete React, Next.js & TypeScript Projects Course 2025" by John Smilga (Udemy)**
25+ projects with TypeScript, includes Next.js, Prisma, Zod, ShadcnUI, TanStack Query, Redux Toolkit.
→ https://www.udemy.com/course/react-tutorial-and-projects-course/

**YouTube (Free):**
- "React in 100 Seconds + Full Tutorial" by Fireship
- "React Tutorial for Beginners" by Net Ninja

### AI SDK / Multi-Agent Patterns

**Vercel AI SDK** — TypeScript-first SDK for calling multiple AI providers (OpenAI, Anthropic, Google) with unified API. Streaming, tool calling, structured output.
→ https://sdk.vercel.ai/docs

**LangChain.js** — Agent framework with tool calling, chains, memory. Good for understanding patterns even if you build custom.
→ https://js.langchain.com/docs/

**YouTube:**
- "Building AI Agents from Scratch" by Dave Ebbelaar — multi-agent patterns
- "AI SDK by Vercel" tutorials — Search: `Vercel AI SDK tutorial`

### Learning Schedule

| Week | Learning Focus | Orquestra Phase |
|---|---|---|
| 1 | Bun + TypeScript courses | — |
| 2 | React + AI SDK docs | Phase 1 (Foundation) |
| 3 | Build core agent system | Phase 2 (Agent Engine) |
| 4-5 | Task scheduler + scrum engine | Phase 3-4 |
| 6-7 | Frontend + dashboard | Phase 5 |
| 8 | Polish + deploy | Phase 6 |

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Runtime | Bun 1.1+ | Fast startup, native TS, built-in SQLite, test runner |
| Language | TypeScript (end-to-end) | Shared types frontend/backend, AI SDKs are TS-first |
| Web Framework | Hono (on Bun) | Lightweight, fast, middleware, WebSocket support |
| Database | PostgreSQL | Reliable, JSONB for agent configs, full-text search |
| ORM | Drizzle ORM | Type-safe, lightweight, great PostgreSQL support |
| Queue/Scheduler | BullMQ + Redis | Scheduled tasks, recurring jobs, retry logic |
| AI Providers | Vercel AI SDK | Unified interface: OpenAI, Anthropic, Google, etc. |
| Real-time | WebSocket (Hono) | Live scrum, streaming agent responses |
| Frontend | React 18 + Vite + TypeScript | Component-based, rich ecosystem |
| UI Library | ShadcnUI + Tailwind CSS | Polished components, utility styling |
| State | TanStack Query + Zustand | Server state + client state |
| File Storage | Local filesystem + S3-compatible | Company artifacts (docs, images) |
| Auth | Better-Auth or Lucia | Modern TS-first auth |
| Monitoring | Built-in token tracking | Per-agent cost accounting |

---

## How the System Works

### Agent Lifecycle

```
1. CREATE AGENT → Define: name, role, avatar, system prompt, AI model,
   API key, tools enabled, temperature, max tokens, budget limit

2. ASSIGN TASKS → Create tasks (one-time or recurring/cron schedule)
   Each task has: description, expected output type, tools allowed,
   priority, deadline, artifact references

3. EXECUTE → Scheduler triggers task at scheduled time
   Agent receives: system prompt + task description + relevant artifacts + tool access
   Agent runs autonomously → produces output (text, document, code, API call results)
   Token usage tracked per execution

4. DAILY SCRUM → At configured time, scrum engine runs:
   - Each agent summarizes: what I did, what I'm doing next, blockers
   - Other agents can ask questions based on the summaries
   - Full transcript saved, you review later

5. REVIEW → You see: task results, scrum transcripts, token costs,
   success/failure rates. Approve, reject, or give feedback.

6. CHAT → Anytime, open a direct conversation with any agent.
   Agent has full context: its role, recent tasks, company artifacts.
```

### Multi-Model Architecture

Each agent can use a different AI provider/model:

```
Agent "Researcher"  → Claude Sonnet (Anthropic)  → API Key A
Agent "Developer"   → GPT-4o (OpenAI)            → API Key B
Agent "Analyst"     → Gemini Pro (Google)         → API Key C
Agent "Writer"      → Claude Haiku (Anthropic)    → API Key A (shared key, different model)
```

The Vercel AI SDK provides a unified interface — you swap models by changing a config, not code.

### Scrum Simulation

The scrum engine is a multi-turn conversation orchestrated by a "Scrum Master" meta-prompt:

```
1. System loads all active agents and their recent task history
2. For each agent, generate standup message:
   - Yesterday: summarize completed tasks
   - Today: summarize upcoming scheduled tasks
   - Blockers: analyze if any task dependencies are unmet
3. All standup messages compiled into a shared context
4. Each agent gets to "read" others' standups and ask one question
5. Addressed agents respond to questions
6. Full transcript saved with timestamp
```

---

## Project Structure

```
orquestra/
├── packages/
│   ├── api/                              ← Bun + Hono backend
│   │   ├── src/
│   │   │   ├── index.ts                  ← App entry, Hono setup
│   │   │   ├── config.ts                 ← Environment + settings
│   │   │   ├── db/
│   │   │   │   ├── schema.ts             ← Drizzle schema definitions
│   │   │   │   ├── migrate.ts            ← Migration runner
│   │   │   │   └── seed.ts               ← Initial data
│   │   │   ├── routes/
│   │   │   │   ├── auth.ts
│   │   │   │   ├── agents.ts
│   │   │   │   ├── tasks.ts
│   │   │   │   ├── scrums.ts
│   │   │   │   ├── conversations.ts
│   │   │   │   ├── artifacts.ts
│   │   │   │   ├── analytics.ts
│   │   │   │   └── webhooks.ts
│   │   │   ├── engine/
│   │   │   │   ├── agent-runner.ts        ← Core: execute agent with context
│   │   │   │   ├── task-executor.ts       ← Run a task through an agent
│   │   │   │   ├── scrum-engine.ts        ← Orchestrate daily scrum
│   │   │   │   ├── tool-registry.ts       ← Available tools per agent
│   │   │   │   └── prompt-builder.ts      ← Assemble system + context prompts
│   │   │   ├── tools/
│   │   │   │   ├── web-search.ts          ← Search the web
│   │   │   │   ├── web-fetch.ts           ← Fetch URL content
│   │   │   │   ├── file-read.ts           ← Read company artifacts
│   │   │   │   ├── file-write.ts          ← Create/update artifacts
│   │   │   │   ├── api-call.ts            ← Call external APIs
│   │   │   │   ├── code-exec.ts           ← Run code in sandbox
│   │   │   │   └── doc-generate.ts        ← Generate documents
│   │   │   ├── scheduler/
│   │   │   │   ├── queue.ts               ← BullMQ queue setup
│   │   │   │   ├── workers.ts             ← Task execution workers
│   │   │   │   └── cron.ts                ← Recurring task scheduler
│   │   │   ├── ai/
│   │   │   │   ├── providers.ts           ← Vercel AI SDK provider setup
│   │   │   │   ├── token-tracker.ts       ← Track usage per agent
│   │   │   │   └── streaming.ts           ← SSE/WebSocket streaming
│   │   │   ├── middleware/
│   │   │   │   ├── auth.ts
│   │   │   │   ├── rate-limit.ts
│   │   │   │   └── error-handler.ts
│   │   │   └── ws/
│   │   │       └── chat.ts               ← WebSocket for live chat + scrum
│   │   ├── package.json
│   │   └── tsconfig.json
│   ├── web/                              ← React frontend
│   │   ├── src/
│   │   │   ├── main.tsx
│   │   │   ├── App.tsx
│   │   │   ├── api/
│   │   │   │   └── client.ts             ← API client + TanStack Query hooks
│   │   │   ├── stores/
│   │   │   │   ├── auth.store.ts
│   │   │   │   └── ui.store.ts
│   │   │   ├── components/
│   │   │   │   ├── layout/
│   │   │   │   │   ├── AppShell.tsx
│   │   │   │   │   ├── Sidebar.tsx
│   │   │   │   │   └── Header.tsx
│   │   │   │   ├── agents/
│   │   │   │   │   ├── AgentCard.tsx
│   │   │   │   │   ├── AgentForm.tsx
│   │   │   │   │   ├── AgentAvatar.tsx
│   │   │   │   │   └── AgentStatus.tsx
│   │   │   │   ├── tasks/
│   │   │   │   │   ├── TaskCard.tsx
│   │   │   │   │   ├── TaskForm.tsx
│   │   │   │   │   ├── TaskTimeline.tsx
│   │   │   │   │   └── TaskResult.tsx
│   │   │   │   ├── scrum/
│   │   │   │   │   ├── ScrumTranscript.tsx
│   │   │   │   │   ├── ScrumTimeline.tsx
│   │   │   │   │   └── AgentMessage.tsx
│   │   │   │   ├── chat/
│   │   │   │   │   ├── ChatWindow.tsx
│   │   │   │   │   ├── MessageBubble.tsx
│   │   │   │   │   └── ChatInput.tsx
│   │   │   │   ├── artifacts/
│   │   │   │   │   ├── ArtifactBrowser.tsx
│   │   │   │   │   ├── ArtifactUpload.tsx
│   │   │   │   │   └── ArtifactViewer.tsx
│   │   │   │   └── analytics/
│   │   │   │       ├── TokenUsageChart.tsx
│   │   │   │       ├── CostBreakdown.tsx
│   │   │   │       ├── TaskSuccessRate.tsx
│   │   │   │       └── AgentPerformance.tsx
│   │   │   └── pages/
│   │   │       ├── Dashboard.tsx
│   │   │       ├── Agents.tsx
│   │   │       ├── AgentDetail.tsx
│   │   │       ├── Tasks.tsx
│   │   │       ├── TaskDetail.tsx
│   │   │       ├── Scrums.tsx
│   │   │       ├── ScrumDetail.tsx
│   │   │       ├── Chat.tsx
│   │   │       ├── Artifacts.tsx
│   │   │       ├── Analytics.tsx
│   │   │       └── Settings.tsx
│   │   ├── package.json
│   │   └── vite.config.ts
│   └── shared/                           ← Shared TypeScript types
│       ├── types/
│       │   ├── agent.ts
│       │   ├── task.ts
│       │   ├── scrum.ts
│       │   ├── conversation.ts
│       │   ├── artifact.ts
│       │   └── analytics.ts
│       └── package.json
├── docker-compose.yml
├── Makefile
├── bun.lockb
├── package.json                          ← Workspace root
└── README.md
```

## File Descriptions

### Backend — Engine (The Brain)

**`engine/agent-runner.ts`** — Core execution engine. Takes an agent config + prompt + tools, calls the AI provider via Vercel AI SDK, streams response, tracks tokens. Handles retries on provider failures. Returns structured result with output, token count, cost, and duration.

**`engine/task-executor.ts`** — Wraps agent-runner for task execution. Loads task context (description, artifacts, previous results), builds the full prompt via prompt-builder, runs the agent, saves result to DB, updates token tracking, triggers webhooks on completion.

**`engine/scrum-engine.ts`** — Orchestrates daily scrum simulation. Steps: (1) Load all active agents + recent task history. (2) For each agent, call AI to generate standup message. (3) Compile all standups into shared context. (4) For each agent, call AI to generate questions about others' updates. (5) Route questions to addressed agents for responses. (6) Save full transcript. Configurable: scrum time, participants, question rounds.

**`engine/tool-registry.ts`** — Registry of available tools. Each tool has: name, description, input schema (Zod), execute function. Tools are assigned per-agent. When an agent wants to use a tool, the registry validates permissions and executes.

**`engine/prompt-builder.ts`** — Assembles the full prompt sent to the AI. Combines: agent system prompt + role description + relevant artifacts content + task description + conversation history + available tools schema. Manages context window limits by truncating/summarizing older content.

### Backend — Tools

**`tools/web-search.ts`** — Searches the web using SerpAPI or Brave Search API. Returns structured results (title, snippet, URL).

**`tools/web-fetch.ts`** — Fetches and extracts text content from a URL. Handles HTML parsing, removes boilerplate, returns clean text.

**`tools/file-read.ts`** — Reads artifacts from the company knowledge base. Supports text, markdown, PDF extraction, JSON, CSV.

**`tools/file-write.ts`** — Creates or updates artifacts. Saves files to storage with metadata (author agent, timestamp, associated task).

**`tools/api-call.ts`** — Makes HTTP requests to configured external APIs. Agent specifies method, URL, headers, body. Supports pre-configured webhook templates.

**`tools/code-exec.ts`** — Executes generated code in a sandboxed environment (Bun subprocess with timeout + memory limits). Returns stdout, stderr, exit code.

**`tools/doc-generate.ts`** — Generates formatted documents (Markdown → PDF, structured reports). Saves as artifact.

### Backend — Scheduler

**`scheduler/queue.ts`** — BullMQ queue configuration. Queues: task-execution (for scheduled tasks), scrum-daily (for daily scrum), artifact-process (for post-processing uploads).

**`scheduler/workers.ts`** — BullMQ workers that consume from queues. Task worker: loads task + agent, calls task-executor, handles errors. Scrum worker: calls scrum-engine at scheduled time.

**`scheduler/cron.ts`** — Manages recurring tasks. On startup, loads all active recurring tasks from DB, creates BullMQ repeatable jobs with cron expressions. Provides add/remove/update methods for runtime changes.

### Backend — AI

**`ai/providers.ts`** — Configures Vercel AI SDK providers. Loads API keys from agent configs. Creates provider instances: `createOpenAI()`, `createAnthropic()`, `createGoogleGenerativeAI()`. Maps agent model strings to provider + model tuples.

**`ai/token-tracker.ts`** — Middleware that wraps every AI call. Records: agent_id, model, prompt_tokens, completion_tokens, total_tokens, estimated_cost (using per-model pricing table). Accumulates per-agent daily/monthly totals. Enforces budget limits — refuses execution if agent has exceeded its budget.

**`ai/streaming.ts`** — Handles streaming responses from AI providers. Pipes tokens to WebSocket for real-time display in chat UI. Buffers for non-streaming contexts (task execution).

### Backend — Routes

**`routes/agents.ts`** — Full CRUD for agents. POST creates agent with all config (name, role, system prompt, model, API key encrypted, tools, temperature, budget). GET returns agent with stats (tasks completed, tokens used, success rate). PUT updates config. DELETE soft-deletes.

**`routes/tasks.ts`** — CRUD for tasks. POST creates one-time or recurring task assigned to an agent. GET /tasks/{id}/result returns execution result. POST /tasks/{id}/run triggers immediate execution. GET /tasks/{id}/history returns all past executions.

**`routes/scrums.ts`** — GET /scrums returns scrum history (paginated). GET /scrums/{id} returns full transcript. POST /scrums/trigger runs scrum immediately. GET /scrums/schedule returns configured schedule.

**`routes/conversations.ts`** — GET /conversations returns chat history per agent. WebSocket endpoint for live streaming chat.

**`routes/artifacts.ts`** — CRUD for company knowledge base. POST uploads file with metadata (tags, description). GET lists/searches artifacts. GET /{id}/content returns file content.

**`routes/analytics.ts`** — GET /analytics/tokens returns token usage per agent/model/time period. GET /analytics/costs returns cost breakdown. GET /analytics/performance returns task success rates, avg execution time.

### Frontend — Key Components

**`components/agents/AgentCard.tsx`** — Card showing agent avatar, name, role, model, status (idle/running/error), today's token usage. Click to navigate to detail.

**`components/agents/AgentForm.tsx`** — Create/edit agent form. Fields: name, role description, avatar (emoji or upload), system prompt (large textarea with syntax highlighting), model selector (dropdown grouped by provider), API key (masked), tools checkboxes, temperature slider, max tokens, monthly budget.

**`components/tasks/TaskCard.tsx`** — Task summary: title, assigned agent, schedule (one-time or cron), last run status, next run time.

**`components/tasks/TaskResult.tsx`** — Displays task execution result: output content (markdown rendered), token usage, execution time, tools used, artifacts created. Expandable to see full prompt and raw response.

**`components/scrum/ScrumTranscript.tsx`** — Renders the full scrum as a chat-like transcript. Each agent's standup in their avatar color. Questions and responses threaded. Expandable sections for each agent.

**`components/chat/ChatWindow.tsx`** — Real-time chat with an agent. WebSocket streaming. Shows agent typing indicator. Agent has context of its role, recent tasks, and company artifacts. Message history persisted.

**`components/analytics/TokenUsageChart.tsx`** — Recharts line/bar chart. Token usage over time per agent. Toggle daily/weekly/monthly. Stacked by model.

**`components/analytics/CostBreakdown.tsx`** — Pie chart of costs by agent, bar chart by model. Total spend this month vs budget. Projections.

### Frontend — Pages

**`pages/Dashboard.tsx`** — Overview: active agents count, tasks running, today's token usage, recent scrum summary, upcoming tasks, cost alerts.

**`pages/Agents.tsx`** — Grid of AgentCards. Create new agent button. Filter by status, model.

**`pages/AgentDetail.tsx`** — Full agent view: config, task history, conversation history, token usage chart, performance stats, edit/delete.

**`pages/Tasks.tsx`** — Task list with filters (agent, status, schedule type). Calendar view for scheduled tasks. Create task button.

**`pages/Scrums.tsx`** — Timeline of past scrums. Click to read transcript. Configure scrum schedule.

**`pages/Chat.tsx`** — Agent selector + ChatWindow. Like a Slack DM interface. Switch between agents.

**`pages/Artifacts.tsx`** — File browser for company knowledge base. Upload, tag, search. Preview text/markdown/PDF.

**`pages/Analytics.tsx`** — Full analytics dashboard: token usage, costs, performance, budget tracking.

## Database Schema

```sql
-- Users (you, the CEO)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- AI Agents
CREATE TABLE agents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    name VARCHAR(100) NOT NULL,
    role VARCHAR(255) NOT NULL,
    avatar VARCHAR(100),
    system_prompt TEXT NOT NULL,
    provider VARCHAR(50) NOT NULL,           -- openai, anthropic, google
    model VARCHAR(100) NOT NULL,             -- gpt-4o, claude-sonnet-4-20250514, gemini-pro
    api_key_encrypted TEXT NOT NULL,
    temperature REAL DEFAULT 0.7,
    max_tokens INTEGER DEFAULT 4096,
    tools_enabled TEXT[] DEFAULT '{}',       -- ['web-search','file-read','api-call',...]
    monthly_budget_usd REAL,
    is_active BOOLEAN DEFAULT true,
    metadata JSONB DEFAULT '{}',
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Tasks
CREATE TABLE tasks (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    agent_id UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    description TEXT NOT NULL,
    expected_output VARCHAR(50) DEFAULT 'text',  -- text, document, code, api_result
    tools_allowed TEXT[],                    -- override agent defaults per task
    priority VARCHAR(20) DEFAULT 'medium',   -- low, medium, high, critical
    schedule_type VARCHAR(20) NOT NULL,      -- once, recurring
    scheduled_at TIMESTAMPTZ,                -- for one-time tasks
    cron_expression VARCHAR(100),            -- for recurring: '0 9 * * 1-5'
    artifact_ids UUID[],                     -- referenced company artifacts
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Task Executions (each run of a task)
CREATE TABLE task_executions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    agent_id UUID NOT NULL REFERENCES agents(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending', -- pending, running, completed, failed
    input_prompt TEXT,
    output TEXT,
    output_artifacts UUID[],                 -- artifacts created by this execution
    tools_used TEXT[],
    prompt_tokens INTEGER DEFAULT 0,
    completion_tokens INTEGER DEFAULT 0,
    total_tokens INTEGER DEFAULT 0,
    estimated_cost_usd REAL DEFAULT 0,
    duration_ms INTEGER,
    error_message TEXT,
    started_at TIMESTAMPTZ,
    completed_at TIMESTAMPTZ,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Daily Scrums
CREATE TABLE scrums (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    participant_agent_ids UUID[] NOT NULL,
    transcript JSONB NOT NULL,               -- [{agent_id, role, content, type: standup|question|answer}]
    total_tokens INTEGER DEFAULT 0,
    estimated_cost_usd REAL DEFAULT 0,
    scrum_date DATE NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Conversations (direct chat with agents)
CREATE TABLE conversations (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    agent_id UUID NOT NULL REFERENCES agents(id) ON DELETE CASCADE,
    user_id UUID NOT NULL REFERENCES users(id),
    title VARCHAR(255),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

CREATE TABLE messages (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    conversation_id UUID NOT NULL REFERENCES conversations(id) ON DELETE CASCADE,
    role VARCHAR(20) NOT NULL,               -- user, assistant
    content TEXT NOT NULL,
    tools_used TEXT[],
    prompt_tokens INTEGER DEFAULT 0,
    completion_tokens INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Company Artifacts (knowledge base)
CREATE TABLE artifacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    created_by_agent_id UUID REFERENCES agents(id),
    created_by_task_id UUID REFERENCES tasks(id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    file_path VARCHAR(500) NOT NULL,
    file_type VARCHAR(50) NOT NULL,          -- markdown, pdf, text, json, csv, image
    file_size_bytes BIGINT,
    tags TEXT[] DEFAULT '{}',
    content_preview TEXT,                    -- first 500 chars for search
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Token Usage Tracking (daily aggregates)
CREATE TABLE token_usage (
    id SERIAL PRIMARY KEY,
    agent_id UUID NOT NULL REFERENCES agents(id),
    date DATE NOT NULL,
    provider VARCHAR(50) NOT NULL,
    model VARCHAR(100) NOT NULL,
    prompt_tokens BIGINT DEFAULT 0,
    completion_tokens BIGINT DEFAULT 0,
    total_tokens BIGINT DEFAULT 0,
    estimated_cost_usd REAL DEFAULT 0,
    request_count INTEGER DEFAULT 0,
    UNIQUE(agent_id, date, model)
);

-- Scrum Schedule Config
CREATE TABLE scrum_config (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    is_enabled BOOLEAN DEFAULT true,
    cron_expression VARCHAR(100) DEFAULT '0 9 * * 1-5',  -- 9am weekdays
    participant_agent_ids UUID[],
    question_rounds INTEGER DEFAULT 1,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_tasks_agent ON tasks(agent_id);
CREATE INDEX idx_task_exec_task ON task_executions(task_id);
CREATE INDEX idx_task_exec_agent ON task_executions(agent_id);
CREATE INDEX idx_task_exec_status ON task_executions(status);
CREATE INDEX idx_messages_conversation ON messages(conversation_id);
CREATE INDEX idx_messages_created ON messages(created_at);
CREATE INDEX idx_artifacts_tags ON artifacts USING GIN (tags);
CREATE INDEX idx_token_usage_agent_date ON token_usage(agent_id, date);
CREATE INDEX idx_scrums_date ON scrums(scrum_date);
```

---

## Configuration

### Environment Variables

```env
# Database
DATABASE_URL=postgresql://orquestra:changeme@localhost:5432/orquestra

# Redis (for BullMQ scheduler)
REDIS_URL=redis://localhost:6379

# Auth
JWT_SECRET=change-this-to-a-long-random-string
ENCRYPTION_KEY=32-byte-hex-key-for-api-key-encryption

# Storage
ARTIFACTS_PATH=./data/artifacts
MAX_ARTIFACT_SIZE_MB=50

# Scrum defaults
DEFAULT_SCRUM_CRON=0 9 * * 1-5
DEFAULT_QUESTION_ROUNDS=1

# Optional: default AI keys (agents can override)
OPENAI_API_KEY=
ANTHROPIC_API_KEY=
GOOGLE_AI_API_KEY=

# Web Search tool
SERP_API_KEY=
```

### docker-compose.yml

```yaml
version: '3.8'
services:
  db:
    image: postgres:16
    environment:
      POSTGRES_DB: orquestra
      POSTGRES_USER: orquestra
      POSTGRES_PASSWORD: changeme
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  api:
    build: ./packages/api
    ports:
      - "3000:3000"
    depends_on:
      - db
      - redis
    environment:
      - DATABASE_URL=postgresql://orquestra:changeme@db:5432/orquestra
      - REDIS_URL=redis://redis:6379
volumes:
  pgdata:
```

---

## API Surface

### Auth
| Method | Endpoint | Description |
|---|---|---|
| POST | /api/auth/register | Create account |
| POST | /api/auth/login | Returns JWT |
| GET | /api/auth/me | Current user |

### Agents
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/agents | List all agents |
| POST | /api/agents | Create agent |
| GET | /api/agents/:id | Agent detail + stats |
| PUT | /api/agents/:id | Update agent config |
| DELETE | /api/agents/:id | Soft delete agent |
| GET | /api/agents/:id/executions | Agent's execution history |
| GET | /api/agents/:id/tokens | Agent's token usage |

### Tasks
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/tasks | List tasks (filterable) |
| POST | /api/tasks | Create task |
| GET | /api/tasks/:id | Task detail |
| PUT | /api/tasks/:id | Update task |
| DELETE | /api/tasks/:id | Delete task |
| POST | /api/tasks/:id/run | Trigger immediate execution |
| GET | /api/tasks/:id/executions | Execution history |
| GET | /api/tasks/:id/executions/:execId | Single execution result |

### Scrums
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/scrums | Scrum history |
| GET | /api/scrums/:id | Full transcript |
| POST | /api/scrums/trigger | Run scrum now |
| GET | /api/scrums/config | Get schedule config |
| PUT | /api/scrums/config | Update schedule |

### Conversations
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/conversations | List conversations |
| POST | /api/conversations | Start new with agent |
| GET | /api/conversations/:id/messages | Message history |
| WS | /ws/chat/:conversationId | Live streaming chat |

### Artifacts
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/artifacts | List/search artifacts |
| POST | /api/artifacts | Upload artifact |
| GET | /api/artifacts/:id | Artifact metadata |
| GET | /api/artifacts/:id/content | Download/view content |
| PUT | /api/artifacts/:id | Update metadata |
| DELETE | /api/artifacts/:id | Delete artifact |

### Analytics
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/analytics/tokens?period=&agent= | Token usage |
| GET | /api/analytics/costs?period= | Cost breakdown |
| GET | /api/analytics/performance | Task success rates |
| GET | /api/analytics/budget | Budget vs actual |

## Token Pricing Table

Built into `ai/token-tracker.ts` for cost estimation:

```typescript
const PRICING: Record<string, {input: number, output: number}> = {
  // OpenAI (per 1M tokens)
  'gpt-4o':           { input: 2.50,  output: 10.00 },
  'gpt-4o-mini':      { input: 0.15,  output: 0.60  },
  'gpt-4-turbo':      { input: 10.00, output: 30.00 },
  'o1':               { input: 15.00, output: 60.00 },
  // Anthropic
  'claude-opus-4-5':  { input: 15.00, output: 75.00 },
  'claude-sonnet-4-5':{ input: 3.00,  output: 15.00 },
  'claude-haiku-4-5': { input: 0.80,  output: 4.00  },
  // Google
  'gemini-2.0-flash': { input: 0.10,  output: 0.40  },
  'gemini-2.0-pro':   { input: 1.25,  output: 5.00  },
};
```

Costs are tracked per execution and aggregated daily per agent.

---

## Scrum Engine Detail

### Prompt Template for Standup

```
You are {agent.name}, a {agent.role} at {company_name}.

Your recent task activity:
{last_24h_executions_summary}

Your upcoming scheduled tasks:
{next_24h_tasks_summary}

Generate your daily standup update in this format:
- **Yesterday:** What you completed (based on actual task results)
- **Today:** What you plan to work on (based on upcoming tasks)
- **Blockers:** Any issues, missing information, or dependencies

Be concise and specific. Reference actual task names and results.
```

### Prompt Template for Questions

```
You are {agent.name}, a {agent.role}.

Here are the standup updates from your colleagues:
{all_standup_messages}

Based on your role and expertise, do you have any questions
or suggestions for your colleagues? Focus on:
- Dependencies that affect your work
- Opportunities to help or collaborate
- Concerns about approach or timeline

Ask at most 1 question. If nothing relevant, say "No questions."
```

### Transcript Format (stored as JSONB)

```json
[
  {"agent_id": "uuid", "agent_name": "Researcher", "type": "standup",
   "content": "**Yesterday:** Completed market analysis for Q1...\n**Today:** Starting competitor deep-dive...\n**Blockers:** None"},
  {"agent_id": "uuid", "agent_name": "Developer", "type": "standup",
   "content": "**Yesterday:** Finished API endpoint for user auth...\n**Today:** Building dashboard components...\n**Blockers:** Waiting on design specs from Writer"},
  {"agent_id": "uuid", "agent_name": "Developer", "type": "question",
   "to_agent_id": "uuid", "to_agent_name": "Researcher",
   "content": "Your market analysis mentioned 3 competitors — could you share the API documentation links you found? I need them for the integration work."},
  {"agent_id": "uuid", "agent_name": "Researcher", "type": "answer",
   "content": "Sure! I saved them as artifacts: competitor-api-docs-v1.md. The key endpoints are..."}
]
```

---

## Offline Task Execution Flow

```
Scheduler fires (BullMQ cron) → Worker picks up job
→ Load task + agent config from DB
→ Build prompt (prompt-builder):
    1. Agent system prompt
    2. Task description
    3. Referenced artifacts (loaded and injected as context)
    4. Tool descriptions (for function calling)
    5. Previous execution result (if recurring, for continuity)
→ Call AI provider (Vercel AI SDK):
    - Use agent's model + API key
    - Enable tool calling if tools are allowed
    - Stream response (buffer for tasks, pipe for chat)
→ If tool call requested:
    - Validate tool is in agent's allowed list
    - Execute tool (web-search, file-read, etc.)
    - Feed result back to AI for next turn
    - Loop until AI produces final response
→ Save execution result to task_executions
→ Update token_usage daily aggregate
→ If output is a document → save as artifact
→ Check budget → warn if approaching limit
→ Mark task execution as completed/failed
```

---

## Implementation Tasks (56 tasks, ~40 days)

### Phase 1 — Foundation (Days 1-4)
1. Initialize Bun workspace (monorepo: api, web, shared)
2. Set up PostgreSQL + Redis (Docker)
3. Define Drizzle schema for all tables
4. Run migrations, write seed data
5. Implement auth routes (register, login, JWT)
6. Set up Hono with middleware (auth, error handler, CORS)
7. Define shared TypeScript types in packages/shared

### Phase 2 — Agent System (Days 5-9)
8. Implement agents CRUD routes
9. API key encryption/decryption (AES-256)
10. Implement ai/providers.ts (Vercel AI SDK multi-provider setup)
11. Implement ai/token-tracker.ts with pricing table
12. Implement engine/prompt-builder.ts
13. Implement engine/agent-runner.ts (core AI execution)
14. Test: create agent, run simple prompt, verify token tracking

### Phase 3 — Tools (Days 10-14)
15. Implement tool-registry.ts with Zod schemas
16. Implement tools/web-search.ts (SerpAPI integration)
17. Implement tools/web-fetch.ts (URL content extraction)
18. Implement tools/file-read.ts (artifact reader)
19. Implement tools/file-write.ts (artifact creator)
20. Implement tools/api-call.ts (HTTP caller)
21. Implement tools/code-exec.ts (sandboxed Bun subprocess)
22. Implement tools/doc-generate.ts
23. Test: agent with tools, verify function calling loop works

### Phase 4 — Task Scheduler (Days 15-19)
24. Set up BullMQ queues + workers
25. Implement task CRUD routes
26. Implement task-executor.ts (prompt assembly + execution + save)
27. Implement cron.ts (recurring task management)
28. Implement immediate task trigger endpoint
29. Test: schedule task, verify it runs at correct time, check result

### Phase 5 — Scrum Engine (Days 20-23)
30. Implement scrum-engine.ts (standup generation)
31. Implement question generation round
32. Implement answer routing
33. Implement scrum CRUD routes + schedule config
34. Set up scrum as BullMQ scheduled job
35. Test: trigger scrum with 3 agents, verify transcript

### Phase 6 — Chat (Days 24-26)
36. Implement conversations CRUD routes
37. Implement WebSocket chat endpoint
38. Implement ai/streaming.ts (SSE for chat)
39. Test: live chat with agent, verify context includes role + artifacts

### Phase 7 — Artifacts + Analytics (Days 27-29)
40. Implement artifacts CRUD routes + file upload
41. Implement artifact content serving + preview
42. Implement analytics routes (token usage, costs, performance)
43. Test: upload artifact, reference in task, verify agent can read it

### Phase 8 — Frontend (Days 30-37)
44. Scaffold React + Vite + Tailwind + ShadcnUI
45. Build AppShell, Sidebar, Header
46. Build Dashboard page (overview stats)
47. Build Agents page + AgentForm + AgentDetail
48. Build Tasks page + TaskForm + TaskResult
49. Build Scrums page + ScrumTranscript
50. Build Chat page + ChatWindow (WebSocket streaming)
51. Build Artifacts page + upload + viewer
52. Build Analytics page (charts with Recharts)

### Phase 9 — Polish (Days 38-40)
53. Input validation on all endpoints (Zod)
54. Error handling audit
55. Write README.md
56. End-to-end test: create agents, assign tasks, run scrum, chat, check analytics

---

## Dependencies

### Backend (packages/api/package.json)

```
hono                               # Web framework
drizzle-orm + drizzle-kit           # Type-safe ORM + migrations
postgres (pg driver)                # PostgreSQL client
ai + @ai-sdk/openai + @ai-sdk/anthropic + @ai-sdk/google  # Vercel AI SDK
bullmq + ioredis                   # Job queue + scheduler
zod                                # Schema validation
jose                               # JWT handling
@node-rs/argon2                    # Password hashing
dotenv                             # Environment vars
nanoid                             # ID generation
linkedom                           # HTML parsing (for web-fetch tool)
```

### Frontend (packages/web/package.json)

```
react + react-dom + react-router-dom
@tanstack/react-query              # Server state management
zustand                            # Client state
tailwindcss + postcss + autoprefixer
@shadcn/ui components              # UI library
recharts                           # Charts for analytics
lucide-react                       # Icons
date-fns                           # Date formatting
react-markdown + remark-gfm        # Render agent markdown output
@microsoft/fetch-event-source      # SSE client for streaming
```

### Shared (packages/shared/package.json)

```
zod                                # Shared validation schemas
```

---

## Architecture Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Runtime | Bun | Fast, native TS, built-in tools, modern DX |
| Framework | Hono | Lightweight, WebSocket support, Bun-optimized |
| Database | PostgreSQL | JSONB for transcripts, reliable, scalable |
| ORM | Drizzle | Type-safe, lightweight, great PostgreSQL support |
| Scheduler | BullMQ + Redis | Battle-tested job queue, cron support, retries |
| AI SDK | Vercel AI SDK | Unified multi-provider, streaming, tool calling |
| API key storage | AES-256 encrypted in DB | Each agent can have its own key |
| Token tracking | Per-execution + daily aggregates | Granular + fast dashboard queries |
| Scrum | Multi-turn orchestrated prompts | Simulates real conversation flow |
| Tool execution | Registry + Zod schema validation | Type-safe, permissioned per agent |
| Code execution | Sandboxed Bun subprocess | Timeout + memory limits for safety |
| Frontend state | TanStack Query + Zustand | Server state (queries) + client state (UI) |
| Chat | WebSocket + SSE | Real-time streaming tokens to UI |
| Monorepo | Bun workspaces | Shared types, single lockfile |

---

## Quick Reference Links

### Runtime & Framework
- Bun: https://bun.sh/docs
- Hono: https://hono.dev/docs
- Drizzle ORM: https://orm.drizzle.team/docs/overview

### AI SDKs
- Vercel AI SDK: https://sdk.vercel.ai/docs
- OpenAI API: https://platform.openai.com/docs
- Anthropic API: https://docs.anthropic.com/
- Google AI: https://ai.google.dev/docs

### Queue & Scheduler
- BullMQ: https://docs.bullmq.io/
- Redis: https://redis.io/docs/

### Frontend
- React: https://react.dev/
- ShadcnUI: https://ui.shadcn.com/
- TanStack Query: https://tanstack.com/query/latest
- Zustand: https://zustand-demo.pmnd.rs/
- Recharts: https://recharts.org/

### Tools & Utilities
- Zod: https://zod.dev/
- SerpAPI: https://serpapi.com/
- date-fns: https://date-fns.org/

---

## Distributed Architecture — Hub & Nodes

### Overview

Orquestra operates as a hub-and-node system. The Hub is the central server (dashboard, scheduler, scrum engine, database). Nodes are lightweight agent runners deployed on any machine. Agents execute where their Node lives, not on the Hub.

```
┌─────────────────────────────────────────┐
│            ORQUESTRA HUB                │
│  Dashboard, Scheduler, Scrum Engine,    │
│  DB, Token Tracking, Artifact Store     │
│                                         │
│  WebSocket Server (:3000/ws/node)       │
└────┬──────────┬──────────┬──────────────┘
     │          │          │
     ▼          ▼          ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Node A  │ │ Node B  │ │ Node C  │
│ Dev PC  │ │ Server  │ │ Cloud   │
│         │ │         │ │         │
│ Agent:  │ │ Agent:  │ │ Agent:  │
│ Writer  │ │ DevOps  │ │ Analyst │
│ Coder   │ │ Monitor │ │ Researcher│
└─────────┘ └─────────┘ └─────────┘
```

### Node Types

**Local Node** — runs on your dev machine. Agents have access to your local filesystem, local dev tools, local network. Great for developer agents that need to read code repos or run builds.

**Server Node** — runs on a company server. Agents have access to production APIs, internal databases, monitoring tools. Good for DevOps or monitoring agents.

**Cloud Node** — runs on a cloud VM. Agents have dedicated compute, useful for heavy tasks (code execution, document generation). Easily scaled.

**Hub-Embedded** — for simplicity, the Hub can also run agents directly (no separate Node needed). Good for getting started before distributing.

---

### Node Package Structure

```
packages/
├── api/         ← Hub (existing)
├── web/         ← Frontend (existing)
├── shared/      ← Shared types (existing)
└── node/        ← NEW: Agent Node runner
    ├── src/
    │   ├── index.ts              ← Node entry point
    │   ├── config.ts             ← Node config (hub URL, node ID, auth token)
    │   ├── connection.ts         ← WebSocket client to Hub
    │   ├── agent-runtime.ts      ← Local agent execution engine
    │   ├── tool-executor.ts      ← Execute tools locally
    │   ├── heartbeat.ts          ← Periodic health + status reports
    │   ├── local-tools/
    │   │   ├── local-filesystem.ts   ← Read/write files on this machine
    │   │   ├── local-shell.ts        ← Run shell commands on this machine
    │   │   ├── local-network.ts      ← Access services on local network
    │   │   └── local-docker.ts       ← Manage containers on this machine
    │   └── sync/
    │       ├── artifact-cache.ts     ← Cache company artifacts locally
    │       └── result-buffer.ts      ← Buffer results if Hub is unreachable
    ├── package.json
    └── tsconfig.json
```

### Node File Descriptions

**`node/src/index.ts`** — Entry point. Reads config, connects to Hub via WebSocket, registers hosted agents, starts heartbeat, listens for task assignments.

**`node/src/config.ts`** — Node configuration:

```typescript
interface NodeConfig {
  nodeId: string;              // Unique ID for this node
  nodeName: string;            // Human-readable name ("Rafael Dev PC")
  hubUrl: string;              // WebSocket URL to Hub
  authToken: string;           // Token for Hub authentication
  agentIds: string[];          // Which agents this node hosts
  localToolsEnabled: boolean;  // Allow filesystem/shell access
  artifactCachePath: string;   // Local cache for company artifacts
  maxConcurrentTasks: number;  // Parallel execution limit
  heartbeatIntervalMs: number; // Default 30000 (30s)
}
```

**`node/src/connection.ts`** — WebSocket client. Connects to Hub, handles reconnection with exponential backoff. Sends: registration, heartbeats, execution results, token usage. Receives: task assignments, scrum participation requests, chat messages, artifact sync commands.

**`node/src/agent-runtime.ts`** — Same core logic as the Hub's agent-runner, but running locally on the node. Uses the agent's API key (received encrypted from Hub, decrypted locally). Calls AI providers directly from the node machine.

**`node/src/tool-executor.ts`** — Executes tools on the local machine. Combines Hub tools (web-search, api-call) with Node-specific local tools (filesystem, shell).

**`node/src/heartbeat.ts`** — Sends periodic status to Hub: node alive, agents status (idle/running), current task, resource usage (CPU, memory), queue depth.

**`node/src/local-tools/local-filesystem.ts`** — Read/write files on the Node machine. Configurable allowed paths (sandboxed). Agent can access local code repos, config files, log files.

**`node/src/local-tools/local-shell.ts`** — Execute shell commands on the Node machine. Sandboxed with timeout + allowed command list. Agent can run builds, tests, deploy scripts.

**`node/src/local-tools/local-network.ts`** — Access services on the Node's local network. Agent on a server node can hit internal APIs, databases, monitoring endpoints not accessible from the Hub.

**`node/src/sync/artifact-cache.ts`** — Caches company artifacts locally so agents don't need to fetch from Hub every execution. Syncs on changes (Hub pushes invalidation events).

**`node/src/sync/result-buffer.ts`** — If Hub is temporarily unreachable, buffers execution results locally. Replays them when connection is restored. Prevents work loss during network issues.

---

### Hub-Node Protocol (WebSocket Messages)

#### Node → Hub

```typescript
// Registration (on connect)
{ type: 'node:register', nodeId: string, nodeName: string, agentIds: string[],
  capabilities: string[], os: string, bunVersion: string }

// Heartbeat (every 30s)
{ type: 'node:heartbeat', nodeId: string, status: 'idle' | 'busy',
  runningTasks: string[], cpuPercent: number, memoryMb: number }

// Task execution result
{ type: 'task:result', executionId: string, taskId: string, agentId: string,
  status: 'completed' | 'failed', output: string, outputArtifacts: string[],
  toolsUsed: string[], promptTokens: number, completionTokens: number,
  estimatedCostUsd: number, durationMs: number, errorMessage?: string }

// Token usage update
{ type: 'tokens:update', agentId: string, model: string,
  promptTokens: number, completionTokens: number, costUsd: number }

// Scrum standup response
{ type: 'scrum:standup', scrumId: string, agentId: string, content: string,
  tokens: number }

// Scrum question/answer
{ type: 'scrum:message', scrumId: string, agentId: string,
  messageType: 'question' | 'answer', toAgentId?: string, content: string }

// Chat response (streaming)
{ type: 'chat:token', conversationId: string, token: string }
{ type: 'chat:done', conversationId: string, fullContent: string,
  promptTokens: number, completionTokens: number }

// Artifact created by agent
{ type: 'artifact:created', artifact: { name, fileType, content, agentId, taskId } }
```

#### Hub → Node

```typescript
// Task assignment
{ type: 'task:assign', executionId: string, taskId: string, agentId: string,
  agentConfig: AgentConfig, taskDescription: string, artifacts: ArtifactContent[],
  previousResult?: string, toolsAllowed: string[] }

// Scrum participation request
{ type: 'scrum:request', scrumId: string, agentId: string,
  phase: 'standup' | 'questions' | 'answers',
  context: string }  // other standups for question phase

// Chat message from user
{ type: 'chat:message', conversationId: string, agentId: string,
  message: string, history: Message[] }

// Artifact sync
{ type: 'artifact:sync', artifactId: string, action: 'created' | 'updated' | 'deleted',
  content?: string }

// Agent config update
{ type: 'agent:update', agentId: string, config: Partial<AgentConfig> }

// Budget warning
{ type: 'budget:warning', agentId: string, usedUsd: number, limitUsd: number }

// Budget exceeded — stop execution
{ type: 'budget:exceeded', agentId: string }
```

---

### Updated Database Tables

```sql
-- Nodes (registered machines)
CREATE TABLE nodes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    name VARCHAR(100) NOT NULL,
    auth_token_hash VARCHAR(255) NOT NULL,
    os VARCHAR(50),
    bun_version VARCHAR(20),
    capabilities TEXT[] DEFAULT '{}',
    last_heartbeat_at TIMESTAMPTZ,
    status VARCHAR(20) DEFAULT 'offline',  -- online, offline, busy
    cpu_percent REAL,
    memory_mb REAL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Agent-Node assignment (which agent lives where)
CREATE TABLE agent_nodes (
    agent_id UUID NOT NULL REFERENCES agents(id),
    node_id UUID NOT NULL REFERENCES nodes(id),
    is_primary BOOLEAN DEFAULT true,        -- primary execution location
    PRIMARY KEY (agent_id, node_id)
);

CREATE INDEX idx_nodes_status ON nodes(status);
CREATE INDEX idx_nodes_heartbeat ON nodes(last_heartbeat_at);
CREATE INDEX idx_agent_nodes_agent ON agent_nodes(agent_id);
CREATE INDEX idx_agent_nodes_node ON agent_nodes(node_id);
```

Add `node_id` to task_executions:

```sql
ALTER TABLE task_executions ADD COLUMN node_id UUID REFERENCES nodes(id);
```

---

### Updated API Surface

| Method | Endpoint | Description |
|---|---|---|
| GET | /api/nodes | List registered nodes |
| POST | /api/nodes | Register new node (returns auth token) |
| GET | /api/nodes/:id | Node detail + status |
| DELETE | /api/nodes/:id | Deregister node |
| PUT | /api/nodes/:id/agents | Assign agents to node |
| GET | /api/nodes/:id/health | Node health + metrics |
| WS | /ws/node | Node connection endpoint |

---

### CLI for Node

Start a node with a simple command:

```bash
# Install globally
bun install -g @orquestra/node

# Register with Hub (first time)
orquestra-node register --hub https://hub.example.com --name "My Dev PC"
# → Outputs: Node registered. Auth token saved to ~/.orquestra/config.json

# Start the node
orquestra-node start
# → Connects to Hub, registers agents, starts listening for tasks

# Or run inline
orquestra-node start --hub https://hub.example.com --token xxx
```

Config stored at `~/.orquestra/config.json`:

```json
{
  "nodeId": "uuid-generated-on-register",
  "nodeName": "Rafael Dev PC",
  "hubUrl": "wss://hub.example.com/ws/node",
  "authToken": "generated-token",
  "localToolsEnabled": true,
  "artifactCachePath": "~/.orquestra/cache",
  "maxConcurrentTasks": 3
}
```

---

### Updated Implementation Tasks (additional)

Add these to the existing 56 tasks:

### Phase 10 — Distributed Nodes (Days 41-48)
57. Create packages/node project structure
58. Implement node/config.ts + CLI argument parsing
59. Implement node/connection.ts (WebSocket client with reconnection)
60. Implement node/agent-runtime.ts (local agent execution)
61. Implement node/heartbeat.ts
62. Implement Hub WebSocket server (/ws/node) with protocol
63. Implement node registration + auth token flow
64. Implement task routing: Hub decides which node runs a task
65. Implement local-tools/local-filesystem.ts (sandboxed)
66. Implement local-tools/local-shell.ts (sandboxed)
67. Implement artifact-cache.ts (sync from Hub)
68. Implement result-buffer.ts (offline buffering)
69. Update scrum-engine to route standup/question requests to correct nodes
70. Update chat to route through node WebSocket
71. Build Node management UI (node list, status, assign agents)
72. Add node_id tracking to task_executions + analytics
73. Test: Hub + 2 nodes on separate machines, agents distributed, run task + scrum

---

### Updated Architecture Decision

| Decision | Choice | Rationale |
|---|---|---|
| Distribution | Hub + Node WebSocket | Agents execute where they live, Hub orchestrates |
| Node connection | Persistent WebSocket | Bidirectional, real-time, survives NAT/firewalls |
| Offline tolerance | Result buffering | Node keeps working if Hub is briefly unreachable |
| Local tools | Sandboxed filesystem + shell | Agents access local machine resources safely |
| Node auth | Token-based (per-node) | Simple, revocable, one token per machine |
| Agent placement | Explicit assignment | You decide where each agent lives |
