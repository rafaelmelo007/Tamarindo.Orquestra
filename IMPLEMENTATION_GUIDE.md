# Orquestra — Complete Project & Learning Guide

## Project Overview

An AI workforce management platform where you create, manage, and orchestrate autonomous AI agents as employees of a virtual company. Each agent has its own role, AI model, system prompt, tools, and budget. Agents execute scheduled tasks autonomously, participate in simulated daily scrums, share status with each other, access company artifacts, and track token costs — all while you review outcomes as the CEO.

Agents can run centrally on the Hub or be distributed across any machine in your infrastructure via lightweight Bun-based Nodes.

**Repo:** `https://github.com/rafaelmelo007/Tamarindo.Orquestra`

---

## Tech Stack

| Layer | Technology | Why |
|---|---|---|
| Hub Backend | .NET 8 Web API (C#) | Your expertise, enterprise-grade, SignalR built-in |
| Hub Database | PostgreSQL | JSONB for transcripts, reliable, scalable |
| Hub ORM | Entity Framework Core + Npgsql | Mature, type-safe |
| Scheduler | Hangfire | .NET-native job scheduler, cron, dashboard |
| Real-time | SignalR | Hub ↔ Frontend + Hub ↔ Nodes communication |
| Auth | ASP.NET Identity + JWT | Standard .NET auth |
| Node Runtime | Bun + TypeScript | Fast, lightweight, deploy on any machine |
| AI (in Nodes) | Vercel AI SDK (TypeScript) | Unified multi-provider, streaming, tool calling |
| Frontend | React 18 + Vite + TypeScript | Component-based, rich ecosystem |
| UI Library | ShadcnUI + Tailwind CSS | Polished components |
| State | TanStack Query + Zustand | Server + client state |
| File Storage | Local filesystem + S3-compatible | Company artifacts |

### Why This Split?

The Hub is C# because it's your strength and it handles the heavy lifting: scheduling, database, auth, orchestration, analytics. SignalR replaces the need for a separate WebSocket library — it handles Frontend real-time updates AND Node communication through the same infrastructure.

The Nodes are Bun + TypeScript because: (1) AI SDKs are TypeScript-first (Vercel AI SDK, OpenAI SDK, Anthropic SDK), (2) Nodes need to be lightweight and deploy anywhere with zero setup, (3) Bun starts fast and has low overhead.

---

## Learning Path & Resources

### .NET 8 Web API (Hub)

**"Build ASP.NET Core Web API - Scratch to Finish (.NET 8)" by Sameer Saini (Udemy)**
11,000+ students. EF Core, repository pattern, DDD, auth.
→ https://www.udemy.com/course/build-rest-apis-with-aspnet-core-web-api-entity-framework/

**"ASP.NET Core 9 Ultimate Guide" by Trevoir Williams (Udemy)**
Clean Architecture, EF Core, JWT, testing, Azure deployment.
→ https://www.udemy.com/course/new-aspnet-core-8-complete-course/

**SignalR Docs:**
→ https://learn.microsoft.com/en-us/aspnet/core/signalr/

**Hangfire (scheduler):**
→ https://www.hangfire.io/

### Bun + TypeScript (Nodes)

**"Bun - The Complete Guide To Fullstack Development" (Udemy)**
Practical, covers Bun APIs, server, bundler, database.
→ https://www.udemy.com/course/bun-the-guide-to-full-stack-dev-with-typescript-and-javascript/

**"Understanding TypeScript - 2025 Edition" by Schwarzmüller (Udemy)**
200,000+ students. Basics through advanced.
→ https://www.udemy.com/course/understanding-typescript/

**Bun Docs:** https://bun.sh/docs

### React (Frontend)

**"Complete React, Next.js & TypeScript Projects Course 2025" by John Smilga (Udemy)**
25+ projects with TypeScript, ShadcnUI, TanStack Query.
→ https://www.udemy.com/course/react-tutorial-and-projects-course/

### AI SDKs

**Vercel AI SDK** → https://sdk.vercel.ai/docs
**OpenAI API** → https://platform.openai.com/docs
**Anthropic API** → https://docs.anthropic.com/

### Learning Schedule

| Week | Learning Focus | Orquestra Phase |
|---|---|---|
| 1 | .NET Web API refresh + Hangfire | Phase 1 (Hub Foundation) |
| 2 | Bun + AI SDK | Phase 2 (Node Runner) |
| 3 | React + ShadcnUI | Phase 3 (Agent System) |
| 4-5 | Build core features | Phase 4-5 (Tasks + Scrum) |
| 6-7 | Frontend + distributed | Phase 6-7 |
| 8 | Polish + deploy | Phase 8 |

---

## How the System Works

### Agent Lifecycle

```
1. CREATE AGENT → Define: name, role, avatar, system prompt, AI model,
   API key, tools, temperature, max tokens, budget limit, assigned node

2. ASSIGN TASKS → One-time or recurring (cron). Each task has: description,
   expected output, tools allowed, priority, deadline, artifact references

3. EXECUTE → Hangfire triggers task → Hub routes to correct Node via SignalR
   → Node runs agent with AI provider → returns result + tokens
   → Hub stores result + updates analytics

4. DAILY SCRUM → Hangfire triggers at configured time:
   - Hub asks each agent's Node for standup → collects responses
   - Hub asks each agent's Node for questions → routes answers
   - Full transcript saved

5. REVIEW → Dashboard: task results, scrum transcripts, token costs,
   success rates. Approve, reject, give feedback.

6. CHAT → Open conversation with any agent. SignalR streams tokens
   from Node through Hub to your browser in real-time.
```

### Distributed Architecture

```
┌─────────────────────────────────────────┐
│          ORQUESTRA HUB (.NET 8)         │
│  API, Hangfire Scheduler, SignalR,      │
│  PostgreSQL, Token Tracking, Artifacts  │
│                                         │
│  SignalR Hubs:                          │
│    /hubs/dashboard  (Frontend)          │
│    /hubs/nodes      (Node connections)  │
└────┬──────────┬──────────┬──────────────┘
     │          │          │
     ▼          ▼          ▼
┌─────────┐ ┌─────────┐ ┌─────────┐
│ Node A  │ │ Node B  │ │ Node C  │
│ Bun+TS  │ │ Bun+TS  │ │ Bun+TS  │
│ Dev PC  │ │ Server  │ │ Cloud   │
│         │ │         │ │         │
│ Writer  │ │ DevOps  │ │ Analyst │
│ Coder   │ │ Monitor │ │ Research│
└─────────┘ └─────────┘ └─────────┘
```

### Multi-Model Per Agent

```
Agent "Researcher"  → Claude Sonnet (Anthropic)  → Key A → Node B
Agent "Developer"   → GPT-4o (OpenAI)            → Key B → Node A
Agent "Analyst"     → Gemini Pro (Google)         → Key C → Node C
Agent "Writer"      → Claude Haiku (Anthropic)    → Key A → Node A
```

---

## Project Structure

```
orquestra/
├── src/
│   ├── Orquestra.Api/                     ← .NET 8 Web API (Hub)
│   │   ├── Program.cs
│   │   ├── appsettings.json
│   │   ├── Controllers/
│   │   │   ├── AuthController.cs
│   │   │   ├── AgentsController.cs
│   │   │   ├── TasksController.cs
│   │   │   ├── ScrumsController.cs
│   │   │   ├── ConversationsController.cs
│   │   │   ├── ArtifactsController.cs
│   │   │   ├── NodesController.cs
│   │   │   └── AnalyticsController.cs
│   │   ├── Hubs/
│   │   │   ├── DashboardHub.cs            ← Frontend real-time updates
│   │   │   └── NodeHub.cs                 ← Node connections + task routing
│   │   ├── Middleware/
│   │   │   ├── ExceptionMiddleware.cs
│   │   │   └── NodeAuthMiddleware.cs
│   │   └── Jobs/
│   │       ├── TaskExecutionJob.cs        ← Hangfire: route task to node
│   │       ├── ScrumJob.cs                ← Hangfire: orchestrate daily scrum
│   │       └── TokenAggregationJob.cs     ← Hangfire: daily token rollup
│   ├── Orquestra.Core/                    ← Domain entities + interfaces
│   │   ├── Entities/
│   │   │   ├── User.cs
│   │   │   ├── Agent.cs
│   │   │   ├── AgentTask.cs
│   │   │   ├── TaskExecution.cs
│   │   │   ├── Scrum.cs
│   │   │   ├── Conversation.cs
│   │   │   ├── Message.cs
│   │   │   ├── Artifact.cs
│   │   │   ├── Node.cs
│   │   │   └── TokenUsage.cs
│   │   ├── Interfaces/
│   │   │   ├── IAgentRepository.cs
│   │   │   ├── ITaskRepository.cs
│   │   │   ├── IScrumService.cs
│   │   │   ├── INodeManager.cs
│   │   │   └── ITokenTracker.cs
│   │   ├── DTOs/
│   │   │   ├── AgentDto.cs
│   │   │   ├── TaskDto.cs
│   │   │   ├── ScrumDto.cs
│   │   │   ├── NodeDto.cs
│   │   │   └── AnalyticsDto.cs
│   │   └── Enums/
│   │       ├── TaskStatus.cs
│   │       ├── NodeStatus.cs
│   │       └── AiProvider.cs
│   ├── Orquestra.Infrastructure/          ← EF Core, services
│   │   ├── Data/
│   │   │   ├── AppDbContext.cs
│   │   │   ├── Configurations/
│   │   │   └── Migrations/
│   │   ├── Repositories/
│   │   │   ├── AgentRepository.cs
│   │   │   └── TaskRepository.cs
│   │   ├── Services/
│   │   │   ├── NodeManager.cs             ← Track connected nodes, route tasks
│   │   │   ├── ScrumOrchestrator.cs       ← Run scrum across distributed nodes
│   │   │   ├── TokenTracker.cs            ← Aggregate token usage + budget check
│   │   │   ├── ArtifactService.cs         ← File upload/download/sync
│   │   │   ├── EncryptionService.cs       ← API key encryption (AES-256)
│   │   │   └── CostCalculator.cs          ← Per-model pricing table
│   │   └── Seed/
│   │       └── DefaultBadgesSeed.cs
│   └── Orquestra.Tests/
├── node/                                  ← Bun + TypeScript Node runner
│   ├── src/
│   │   ├── index.ts                       ← Entry point + CLI
│   │   ├── config.ts                      ← Node config
│   │   ├── hub-connection.ts              ← SignalR client to Hub
│   │   ├── agent-runtime.ts               ← AI execution engine
│   │   ├── prompt-builder.ts              ← Assemble prompts
│   │   ├── tool-registry.ts               ← Tool management
│   │   ├── heartbeat.ts                   ← Health reporting
│   │   ├── tools/
│   │   │   ├── web-search.ts
│   │   │   ├── web-fetch.ts
│   │   │   ├── file-read.ts              ← Read company artifacts (cached)
│   │   │   ├── file-write.ts             ← Create artifacts → send to Hub
│   │   │   ├── api-call.ts
│   │   │   ├── code-exec.ts              ← Sandboxed Bun subprocess
│   │   │   ├── doc-generate.ts
│   │   │   ├── local-filesystem.ts       ← Read/write on this machine
│   │   │   └── local-shell.ts            ← Run commands on this machine
│   │   ├── ai/
│   │   │   ├── providers.ts              ← Vercel AI SDK setup
│   │   │   └── token-counter.ts          ← Count tokens per call
│   │   └── sync/
│   │       ├── artifact-cache.ts         ← Local artifact cache
│   │       └── result-buffer.ts          ← Buffer if Hub offline
│   ├── package.json
│   └── tsconfig.json
├── web/                                   ← React frontend
│   ├── src/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   ├── api/
│   │   │   └── client.ts
│   │   ├── stores/
│   │   │   ├── auth.store.ts
│   │   │   └── ui.store.ts
│   │   ├── components/
│   │   │   ├── layout/
│   │   │   │   ├── AppShell.tsx
│   │   │   │   ├── Sidebar.tsx
│   │   │   │   └── Header.tsx
│   │   │   ├── agents/
│   │   │   │   ├── AgentCard.tsx
│   │   │   │   ├── AgentForm.tsx
│   │   │   │   └── AgentStatus.tsx
│   │   │   ├── tasks/
│   │   │   │   ├── TaskCard.tsx
│   │   │   │   ├── TaskForm.tsx
│   │   │   │   └── TaskResult.tsx
│   │   │   ├── scrum/
│   │   │   │   ├── ScrumTranscript.tsx
│   │   │   │   └── AgentMessage.tsx
│   │   │   ├── chat/
│   │   │   │   ├── ChatWindow.tsx
│   │   │   │   ├── MessageBubble.tsx
│   │   │   │   └── ChatInput.tsx
│   │   │   ├── nodes/
│   │   │   │   ├── NodeCard.tsx
│   │   │   │   ├── NodeStatus.tsx
│   │   │   │   └── NodeAgentAssign.tsx
│   │   │   ├── artifacts/
│   │   │   │   ├── ArtifactBrowser.tsx
│   │   │   │   └── ArtifactViewer.tsx
│   │   │   └── analytics/
│   │   │       ├── TokenUsageChart.tsx
│   │   │       ├── CostBreakdown.tsx
│   │   │       └── AgentPerformance.tsx
│   │   └── pages/
│   │       ├── Dashboard.tsx
│   │       ├── Agents.tsx
│   │       ├── AgentDetail.tsx
│   │       ├── Tasks.tsx
│   │       ├── Scrums.tsx
│   │       ├── ScrumDetail.tsx
│   │       ├── Chat.tsx
│   │       ├── Nodes.tsx
│   │       ├── Artifacts.tsx
│   │       ├── Analytics.tsx
│   │       └── Settings.tsx
│   ├── package.json
│   └── vite.config.ts
├── docker-compose.yml
├── Makefile
├── Orquestra.sln
└── README.md
```

## File Descriptions

### Hub — Orquestra.Api

**`Controllers/AgentsController.cs`** — CRUD for agents. POST creates with config (name, role, system prompt, model, encrypted API key, tools, temperature, budget). GET returns agent + stats (tasks completed, tokens used, success rate). Includes node assignment.

**`Controllers/TasksController.cs`** — CRUD for tasks. POST creates one-time or recurring (cron). POST /:id/run triggers immediate execution. GET /:id/executions returns history. Hub routes execution to the agent's assigned Node.

**`Controllers/ScrumsController.cs`** — GET returns history. GET /:id returns transcript. POST /trigger runs scrum now. PUT /config updates schedule. Hub orchestrates across Nodes.

**`Controllers/ConversationsController.cs`** — GET lists conversations per agent. SignalR used for live streaming chat (Hub relays between browser and Node).

**`Controllers/NodesController.cs`** — Node registration, status, agent assignment. GET returns all nodes with health status. POST registers new node (generates auth token). PUT /:id/agents assigns agents to a node.

**`Controllers/AnalyticsController.cs`** — Token usage, costs, performance. Filterable by agent, model, date range. Budget vs actual comparison.

**`Hubs/NodeHub.cs`** — SignalR hub for Node connections. Nodes connect with auth token. Hub maintains a registry of connected nodes and their hosted agents. Routes task assignments, scrum requests, and chat messages to the correct Node. Receives execution results and token usage updates.

**`Hubs/DashboardHub.cs`** — SignalR hub for frontend. Pushes real-time updates: task started/completed, new scrum transcript, node status changes, budget alerts, streaming chat tokens.

**`Jobs/TaskExecutionJob.cs`** — Hangfire job. Loads task + agent, finds the agent's Node, sends task assignment via SignalR NodeHub, waits for result. Saves execution to DB. Updates token tracking. Handles timeout if Node doesn't respond.

**`Jobs/ScrumJob.cs`** — Hangfire recurring job. Orchestrates scrum: (1) Send standup request to each participant's Node, (2) Collect responses, (3) Send question phase with all standups as context, (4) Route answers, (5) Save transcript.

**`Jobs/TokenAggregationJob.cs`** — Hangfire daily job. Rolls up per-execution token counts into daily aggregates per agent/model. Checks budgets and flags agents approaching limits.

### Hub — Orquestra.Infrastructure

**`Services/NodeManager.cs`** — Tracks connected Nodes via SignalR connection IDs. Routes messages to correct Node. Handles Node disconnection (marks agents as offline, queues tasks for retry). Provides methods: SendTaskToNode, SendScrumRequest, SendChatMessage.

**`Services/ScrumOrchestrator.cs`** — Multi-step scrum logic. For each participating agent: (1) Find its Node, (2) Send standup request with recent task history, (3) Wait for response, (4) Compile all standups, (5) Send question round, (6) Route answers between Nodes, (7) Assemble and save transcript.

**`Services/TokenTracker.cs`** — Receives token updates from Nodes. Stores per-execution and aggregates daily. Checks budget on every update. If agent exceeds monthly budget, sends budget:exceeded signal to its Node.

**`Services/CostCalculator.cs`** — Per-model pricing table. Calculates estimated cost from token counts. Updated when providers change pricing.

**`Services/EncryptionService.cs`** — AES-256 encryption for API keys stored in DB. Keys encrypted at rest, decrypted only when sending to a Node for execution.

### Node — Bun + TypeScript

**`node/src/index.ts`** — CLI entry point. Commands: `register` (connect to Hub, get auth token), `start` (connect and listen). Reads config from ~/.orquestra/config.json or CLI args.

**`node/src/hub-connection.ts`** — SignalR client (using @microsoft/signalr npm package). Connects to Hub's NodeHub. Handles reconnection with exponential backoff. Registers hosted agents on connect. Listens for task assignments, scrum requests, chat messages.

**`node/src/agent-runtime.ts`** — Core AI execution. Takes agent config + prompt + tools, calls AI provider via Vercel AI SDK. Handles tool calling loop (AI requests tool → execute → feed result back → repeat until final answer). Streams tokens for chat. Returns structured result.

**`node/src/prompt-builder.ts`** — Assembles full prompt: agent system prompt + role + artifacts + task description + conversation history + tools schema. Manages context window by truncating old content.

**`node/src/tool-registry.ts`** — Registry of tools with Zod schemas. Validates agent permissions before executing. Combines Hub tools (web-search, api-call) with local tools (filesystem, shell).

**`node/src/tools/local-filesystem.ts`** — Read/write files on the Node machine. Sandboxed to configured allowed paths. Agent on your dev PC can read code repos.

**`node/src/tools/local-shell.ts`** — Run shell commands. Sandboxed with timeout, memory limits, allowed command list. Agent can run builds, tests, scripts.

**`node/src/sync/artifact-cache.ts`** — Caches company artifacts locally. Hub pushes invalidation events. Agent reads from local cache instead of fetching from Hub every time.

**`node/src/sync/result-buffer.ts`** — If Hub is unreachable, buffers execution results locally. Replays when connection restores.

### Frontend — React

**`pages/Dashboard.tsx`** — Overview: active agents, running tasks, today's tokens, recent scrum summary, node health, cost alerts.

**`pages/Agents.tsx`** — Grid of AgentCards. Each shows name, role, model, node assignment, status, today's tokens. Create/edit agent form.

**`pages/AgentDetail.tsx`** — Full agent view: config, task history, conversations, token chart, performance, assigned node, edit/delete.

**`pages/Tasks.tsx`** — Task list filterable by agent, status, schedule. Calendar view for scheduled tasks. Create task with cron builder.

**`pages/Scrums.tsx`** — Timeline of daily scrums. Click to read transcript. Configure schedule + participants.

**`pages/ScrumDetail.tsx`** — Full scrum transcript rendered as chat. Each agent's standup, questions, and answers. Token cost for the scrum.

**`pages/Chat.tsx`** — Agent selector + ChatWindow. Real-time streaming via SignalR. Agent has role context, recent tasks, artifacts.

**`pages/Nodes.tsx`** — Node management. List of registered nodes with status (online/offline/busy), CPU/memory, hosted agents. Register new node (shows CLI command). Assign/reassign agents.

**`pages/Artifacts.tsx`** — Company knowledge base browser. Upload, tag, search. Preview text/markdown/PDF. See which agents created/referenced each artifact.

**`pages/Analytics.tsx`** — Token usage charts per agent/model, cost breakdown, budget tracking, task success rates, agent performance comparison.

---

## Database Schema

```sql
-- Users (the CEO)
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    name VARCHAR(100),
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Nodes (machines running agents)
CREATE TABLE nodes (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    name VARCHAR(100) NOT NULL,
    auth_token_hash VARCHAR(255) NOT NULL,
    os VARCHAR(50),
    bun_version VARCHAR(20),
    capabilities TEXT[] DEFAULT '{}',
    last_heartbeat_at TIMESTAMPTZ,
    status VARCHAR(20) DEFAULT 'offline',
    cpu_percent REAL,
    memory_mb REAL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- AI Agents
CREATE TABLE agents (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    node_id UUID REFERENCES nodes(id),
    name VARCHAR(100) NOT NULL,
    role VARCHAR(255) NOT NULL,
    avatar VARCHAR(100),
    system_prompt TEXT NOT NULL,
    provider VARCHAR(50) NOT NULL,
    model VARCHAR(100) NOT NULL,
    api_key_encrypted TEXT NOT NULL,
    temperature REAL DEFAULT 0.7,
    max_tokens INTEGER DEFAULT 4096,
    tools_enabled TEXT[] DEFAULT '{}',
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
    expected_output VARCHAR(50) DEFAULT 'text',
    tools_allowed TEXT[],
    priority VARCHAR(20) DEFAULT 'medium',
    schedule_type VARCHAR(20) NOT NULL,
    scheduled_at TIMESTAMPTZ,
    cron_expression VARCHAR(100),
    artifact_ids UUID[],
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Task Executions
CREATE TABLE task_executions (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    task_id UUID NOT NULL REFERENCES tasks(id) ON DELETE CASCADE,
    agent_id UUID NOT NULL REFERENCES agents(id),
    node_id UUID REFERENCES nodes(id),
    status VARCHAR(20) NOT NULL DEFAULT 'pending',
    input_prompt TEXT,
    output TEXT,
    output_artifacts UUID[],
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

-- Scrums
CREATE TABLE scrums (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID NOT NULL REFERENCES users(id),
    participant_agent_ids UUID[] NOT NULL,
    transcript JSONB NOT NULL,
    total_tokens INTEGER DEFAULT 0,
    estimated_cost_usd REAL DEFAULT 0,
    scrum_date DATE NOT NULL,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Conversations
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
    role VARCHAR(20) NOT NULL,
    content TEXT NOT NULL,
    tools_used TEXT[],
    prompt_tokens INTEGER DEFAULT 0,
    completion_tokens INTEGER DEFAULT 0,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Artifacts
CREATE TABLE artifacts (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id UUID REFERENCES users(id),
    created_by_agent_id UUID REFERENCES agents(id),
    created_by_task_id UUID REFERENCES tasks(id),
    name VARCHAR(255) NOT NULL,
    description TEXT,
    file_path VARCHAR(500) NOT NULL,
    file_type VARCHAR(50) NOT NULL,
    file_size_bytes BIGINT,
    tags TEXT[] DEFAULT '{}',
    content_preview TEXT,
    is_active BOOLEAN DEFAULT true,
    created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Token Usage (daily aggregates)
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

-- Scrum Config
CREATE TABLE scrum_config (
    id SERIAL PRIMARY KEY,
    user_id UUID NOT NULL REFERENCES users(id),
    is_enabled BOOLEAN DEFAULT true,
    cron_expression VARCHAR(100) DEFAULT '0 9 * * 1-5',
    participant_agent_ids UUID[],
    question_rounds INTEGER DEFAULT 1
);

-- Gamification
CREATE TABLE badges (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    description TEXT,
    icon VARCHAR(50),
    criteria JSONB NOT NULL
);

CREATE TABLE user_badges (
    user_id UUID NOT NULL REFERENCES users(id),
    badge_id INTEGER NOT NULL REFERENCES badges(id),
    earned_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
    PRIMARY KEY (user_id, badge_id)
);

-- Indexes
CREATE INDEX idx_agents_node ON agents(node_id);
CREATE INDEX idx_tasks_agent ON tasks(agent_id);
CREATE INDEX idx_task_exec_task ON task_executions(task_id);
CREATE INDEX idx_task_exec_agent ON task_executions(agent_id);
CREATE INDEX idx_task_exec_node ON task_executions(node_id);
CREATE INDEX idx_task_exec_status ON task_executions(status);
CREATE INDEX idx_messages_conversation ON messages(conversation_id);
CREATE INDEX idx_artifacts_tags ON artifacts USING GIN (tags);
CREATE INDEX idx_token_usage_agent ON token_usage(agent_id, date);
CREATE INDEX idx_nodes_status ON nodes(status);
CREATE INDEX idx_scrums_date ON scrums(scrum_date);
```

## Configuration

### appsettings.json (Hub)

```json
{
  "ConnectionStrings": {
    "DefaultConnection": "Host=localhost;Port=5432;Database=orquestra;Username=orquestra;Password=changeme"
  },
  "Jwt": { "Secret": "change-this", "Issuer": "Orquestra", "ExpiryHours": 168 },
  "Encryption": { "Key": "32-byte-hex-key-for-api-keys" },
  "Storage": { "ArtifactsPath": "./data/artifacts", "MaxSizeMb": 50 },
  "Hangfire": { "DashboardPath": "/hangfire" },
  "Scrum": { "DefaultCron": "0 9 * * 1-5", "QuestionRounds": 1 }
}
```

### Node Config (~/.orquestra/config.json)

```json
{
  "nodeId": "uuid",
  "nodeName": "Rafael Dev PC",
  "hubUrl": "https://hub.example.com",
  "authToken": "generated-token",
  "localToolsEnabled": true,
  "allowedPaths": ["/home/rafael/projects"],
  "allowedCommands": ["npm", "bun", "git", "docker"],
  "artifactCachePath": "~/.orquestra/cache",
  "maxConcurrentTasks": 3,
  "heartbeatIntervalMs": 30000
}
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
    ports: ["5432:5432"]
    volumes: [pgdata:/var/lib/postgresql/data]
  api:
    build: ./src/Orquestra.Api
    ports: ["5000:8080"]
    depends_on: [db]
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

### Agents
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/agents | List all |
| POST | /api/agents | Create |
| GET | /api/agents/:id | Detail + stats |
| PUT | /api/agents/:id | Update config |
| DELETE | /api/agents/:id | Soft delete |
| GET | /api/agents/:id/executions | Execution history |
| GET | /api/agents/:id/tokens | Token usage |

### Tasks
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/tasks | List (filterable) |
| POST | /api/tasks | Create |
| GET | /api/tasks/:id | Detail |
| PUT | /api/tasks/:id | Update |
| POST | /api/tasks/:id/run | Trigger now |
| GET | /api/tasks/:id/executions | History |

### Scrums
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/scrums | History |
| GET | /api/scrums/:id | Transcript |
| POST | /api/scrums/trigger | Run now |
| PUT | /api/scrums/config | Update schedule |

### Conversations
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/conversations | List |
| POST | /api/conversations | Start new |
| GET | /api/conversations/:id/messages | History |

### Nodes
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/nodes | List nodes |
| POST | /api/nodes | Register (returns token) |
| GET | /api/nodes/:id | Detail + health |
| DELETE | /api/nodes/:id | Deregister |
| PUT | /api/nodes/:id/agents | Assign agents |

### Artifacts
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/artifacts | List/search |
| POST | /api/artifacts | Upload |
| GET | /api/artifacts/:id/content | Download |
| DELETE | /api/artifacts/:id | Delete |

### Analytics
| Method | Endpoint | Description |
|---|---|---|
| GET | /api/analytics/tokens | Token usage |
| GET | /api/analytics/costs | Cost breakdown |
| GET | /api/analytics/performance | Success rates |

### Real-time (SignalR)
| Hub | Endpoint | Clients |
|---|---|---|
| DashboardHub | /hubs/dashboard | Frontend browsers |
| NodeHub | /hubs/nodes | Bun Node processes |

---

## Hub-Node Protocol (SignalR Messages)

### Node → Hub

```
node:register     { nodeId, nodeName, agentIds, os, bunVersion }
node:heartbeat    { nodeId, status, runningTasks, cpuPercent, memoryMb }
task:result       { executionId, taskId, agentId, status, output, tokens, cost, duration }
tokens:update     { agentId, model, promptTokens, completionTokens, costUsd }
scrum:standup     { scrumId, agentId, content, tokens }
scrum:message     { scrumId, agentId, type, toAgentId, content }
chat:token        { conversationId, token }
chat:done         { conversationId, fullContent, tokens }
artifact:created  { name, fileType, content, agentId, taskId }
```

### Hub → Node

```
task:assign       { executionId, taskId, agentId, agentConfig, taskDesc, artifacts, tools }
scrum:request     { scrumId, agentId, phase, context }
chat:message      { conversationId, agentId, message, history }
artifact:sync     { artifactId, action, content }
agent:update      { agentId, config }
budget:warning    { agentId, usedUsd, limitUsd }
budget:exceeded   { agentId }
```

---

## Scrum Engine Detail

### Standup Prompt

```
You are {agent.name}, a {agent.role} at the company.

Your recent task activity:
{last_24h_executions_summary}

Upcoming scheduled tasks:
{next_24h_tasks_summary}

Generate your standup:
- **Yesterday:** What you completed
- **Today:** What you plan to work on
- **Blockers:** Any issues or dependencies

Be concise. Reference actual task names and results.
```

### Question Prompt

```
You are {agent.name}, a {agent.role}.

Colleagues' standups:
{all_standup_messages}

Ask at most 1 question relevant to your role. If nothing, say "No questions."
```

### Transcript Format (JSONB)

```json
[
  {"agent_id": "uuid", "agent_name": "Researcher", "type": "standup",
   "content": "**Yesterday:** Completed market analysis..."},
  {"agent_id": "uuid", "agent_name": "Developer", "type": "question",
   "to_agent_id": "uuid", "to_agent_name": "Researcher",
   "content": "Could you share the API docs you found?"},
  {"agent_id": "uuid", "agent_name": "Researcher", "type": "answer",
   "content": "Saved as artifact: competitor-api-docs.md"}
]
```

---

## Token Pricing (in CostCalculator.cs)

```csharp
// Per 1M tokens
{ "gpt-4o",            (Input: 2.50,  Output: 10.00) },
{ "gpt-4o-mini",       (Input: 0.15,  Output: 0.60)  },
{ "claude-opus-4-5",   (Input: 15.00, Output: 75.00) },
{ "claude-sonnet-4-5", (Input: 3.00,  Output: 15.00) },
{ "claude-haiku-4-5",  (Input: 0.80,  Output: 4.00)  },
{ "gemini-2.0-flash",  (Input: 0.10,  Output: 0.40)  },
{ "gemini-2.0-pro",    (Input: 1.25,  Output: 5.00)  },
```

---

## Node CLI

```bash
# Install
bun install -g @orquestra/node

# Register with Hub
orquestra-node register --hub https://hub.example.com --name "My Dev PC"
# → Node registered. Token saved to ~/.orquestra/config.json

# Start
orquestra-node start

# Or inline
orquestra-node start --hub https://hub.example.com --token xxx
```

---

## Implementation Tasks (73 tasks, ~48 days)

### Phase 1 — Hub Foundation (Days 1-5)
1. Create .NET solution (Api, Core, Infrastructure, Tests)
2. Set up PostgreSQL (Docker)
3. Implement EF Core DbContext + migrations
4. Seed data (default badges)
5. Implement auth (ASP.NET Identity + JWT)
6. Set up Hangfire scheduler
7. Implement SignalR hubs (DashboardHub, NodeHub)

### Phase 2 — Agent System (Days 6-10)
8. Implement agents CRUD
9. API key encryption service (AES-256)
10. Implement CostCalculator with pricing table
11. Implement TokenTracker service
12. Implement NodeManager (track connections, route messages)
13. Test: create agent, assign to node

### Phase 3 — Node Runner (Days 11-16)
14. Initialize node/ Bun project
15. Implement config.ts + CLI (register, start)
16. Implement hub-connection.ts (SignalR client)
17. Implement agent-runtime.ts (Vercel AI SDK)
18. Implement prompt-builder.ts
19. Implement tool-registry.ts with Zod schemas
20. Implement tools: web-search, web-fetch, api-call
21. Implement tools: file-read, file-write, doc-generate
22. Implement tools: code-exec (sandboxed Bun subprocess)
23. Implement tools: local-filesystem, local-shell
24. Implement heartbeat.ts
25. Test: Node connects to Hub, receives task, executes, reports result

### Phase 4 — Task Scheduler (Days 17-21)
26. Implement tasks CRUD
27. Implement TaskExecutionJob (Hangfire → route to Node)
28. Implement cron scheduling for recurring tasks
29. Implement immediate trigger endpoint
30. Implement execution result storage + status tracking
31. Test: schedule task, verify Hangfire triggers, Node executes, result saved

### Phase 5 — Scrum Engine (Days 22-26)
32. Implement ScrumOrchestrator service
33. Implement standup generation (route to Nodes, collect responses)
34. Implement question round
35. Implement answer routing between Nodes
36. Implement ScrumJob (Hangfire recurring)
37. Implement scrum config CRUD
38. Test: trigger scrum with 3 agents on 2 nodes, verify transcript

### Phase 6 — Chat + Artifacts (Days 27-30)
39. Implement conversations CRUD
40. Implement SignalR chat relay (browser → Hub → Node → Hub → browser)
41. Implement streaming token relay
42. Implement artifacts CRUD + file upload
43. Implement artifact-cache.ts in Node
44. Implement result-buffer.ts in Node
45. Test: chat with agent, verify streaming; upload artifact, verify agent reads it

### Phase 7 — Frontend (Days 31-39)
46. Scaffold React + Vite + Tailwind + ShadcnUI
47. Build AppShell, Sidebar, Header
48. Build Dashboard page
49. Build Agents page + AgentForm + AgentDetail
50. Build Tasks page + TaskForm + TaskResult
51. Build Scrums page + ScrumTranscript
52. Build Chat page + ChatWindow (SignalR streaming)
53. Build Nodes page + NodeCard + agent assignment
54. Build Artifacts page + upload + viewer
55. Build Analytics page (Recharts charts)
56. Wire SignalR DashboardHub for real-time updates

### Phase 8 — Advanced (Days 40-44)
57. Implement budget enforcement (Hub sends exceeded signal)
58. Implement agent performance scoring
59. Implement artifact search (full-text)
60. Implement task dependencies (task B waits for task A)
61. Implement execution retry on Node failure
62. Implement Node reconnection + result replay

### Phase 9 — Polish (Days 45-48)
63. Input validation (FluentValidation)
64. Error handling audit
65. Rate limiting
66. Hangfire dashboard security
67. Write README.md
68. End-to-end test: Hub + 2 Nodes, create agents, schedule tasks, run scrum, chat, check analytics

---

## Dependencies

### Hub (.NET NuGet)

```
Microsoft.AspNetCore.Authentication.JwtBearer
Microsoft.AspNetCore.Identity.EntityFrameworkCore
Microsoft.AspNetCore.SignalR
Npgsql.EntityFrameworkCore.PostgreSQL
Hangfire + Hangfire.PostgreSql
FluentValidation.AspNetCore
Serilog.AspNetCore
Swashbuckle.AspNetCore
AutoMapper
```

### Node (Bun/npm)

```
@microsoft/signalr          # SignalR client for Hub connection
ai + @ai-sdk/openai + @ai-sdk/anthropic + @ai-sdk/google  # Vercel AI SDK
zod                          # Schema validation
nanoid                       # ID generation
linkedom                     # HTML parsing (web-fetch tool)
commander                    # CLI argument parsing
```

### Frontend (npm)

```
react + react-dom + react-router-dom
@tanstack/react-query
zustand
tailwindcss + @shadcn/ui
recharts
lucide-react
@microsoft/signalr           # SignalR client
react-markdown + remark-gfm
date-fns
```

---

## Architecture Decisions

| Decision | Choice | Rationale |
|---|---|---|
| Hub runtime | .NET 8 | Your expertise, enterprise-grade, SignalR built-in |
| Node runtime | Bun + TypeScript | AI SDKs are TS-first, lightweight, fast startup |
| Hub ↔ Node | SignalR | Bidirectional, .NET native, handles reconnection |
| Scheduler | Hangfire | .NET-native, cron, dashboard, PostgreSQL storage |
| Database | PostgreSQL | JSONB for transcripts, reliable |
| ORM | EF Core | Mature, type-safe, migration support |
| AI SDK | Vercel AI SDK (in Node) | Unified multi-provider, streaming, tool calling |
| Node auth | Token per node | Simple, revocable, generated on registration |
| API keys | AES-256 in DB | Encrypted at rest, decrypted only for execution |
| Token tracking | Per-execution + daily rollup | Granular + fast analytics |
| Frontend state | TanStack Query + Zustand | Server state + client state |
| Chat streaming | SignalR relay | Browser ↔ Hub ↔ Node, real-time tokens |

---

## Quick Reference Links

### Hub
- ASP.NET Core: https://learn.microsoft.com/en-us/aspnet/core/
- EF Core: https://learn.microsoft.com/en-us/ef/core/
- SignalR: https://learn.microsoft.com/en-us/aspnet/core/signalr/
- Hangfire: https://www.hangfire.io/

### Node
- Bun: https://bun.sh/docs
- Vercel AI SDK: https://sdk.vercel.ai/docs
- @microsoft/signalr (JS client): https://www.npmjs.com/package/@microsoft/signalr

### AI Providers
- OpenAI: https://platform.openai.com/docs
- Anthropic: https://docs.anthropic.com/
- Google AI: https://ai.google.dev/docs

### Frontend
- React: https://react.dev/
- ShadcnUI: https://ui.shadcn.com/
- TanStack Query: https://tanstack.com/query/latest
- Recharts: https://recharts.org/
