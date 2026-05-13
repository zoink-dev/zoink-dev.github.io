---
layout: default
title: Zoink — Multi-Framework AI Agent Swarm Orchestration
description: Open-source CLI, terminal dashboard, and web dashboard for running OpenClaw + Hermes AI agent swarms inside Docker.
---

# Zoink

**Multi-Framework AI Agent Swarm Orchestration**

Zoink is an open-source CLI, terminal dashboard, and web dashboard for running AI agent swarms inside Docker. Define your team in `swarm.yaml`, launch it with one command, then manage runtime, health, logs, state, task flow, and chat from the CLI, the TUI, or a browser.

```bash
npm install -g @zoink-dev/zoink-cli
zoink config init
zoink up
```

[View on GitHub »](https://github.com/zoink-dev/zoink-cli) &nbsp;·&nbsp; [npm package »](https://www.npmjs.com/package/@zoink-dev/zoink-cli) &nbsp;·&nbsp;

---

## On this page

- [What is Zoink?](#what-is-zoink)
- [Features](#features)
- [Quick start](#quick-start)
- [Commands](#commands)
- [Dashboards](#dashboards)
- [Daemon & remote Kanban](#daemon--remote-kanban)
- [Runtimes & roles](#runtimes--roles)
- [Requirements](#requirements)

---

## What is Zoink?

Zoink is a **multi-framework control plane** for AI agents. v0.3 evolved it from a single-framework orchestrator into a tool that can run [OpenClaw](https://github.com/openclaw/openclaw) and [Hermes](https://hermes-agent.nousresearch.com/) agents **side-by-side in the same swarm**, sharing one Docker network, one workspace, and one Kanban board.

Each agent runs in its own isolated Docker container with persistent state, health checks, resource limits, and bearer-authenticated HTTP access for chat and config reload.

**Who it's for**

- AI-heavy developers building multi-agent workflows on a local machine
- Platform teams running a shared agent control plane behind a daemon
- OSS contributors exploring runtime-agnostic agent orchestration

---

## Features

### Orchestration

- **Multi-framework swarms** — Mix OpenClaw and Hermes via a per-agent `runtime` field
- **Declarative `swarm.yaml` v2.0** — `frameworks`, `runtime`, `provider`, `image`, `profiles`, `cpu_limit`, `memory_limit`, `gateway`, `ports`, `persistState`, `autoPickup`, `temperature`, `skills`, optional `kanban` block
- **One-command lifecycle** — `zoink up` / `zoink down` handles networks, containers, volumes, and the Kanban board
- **Idempotent operations** — Re-run `zoink up` safely; containers are matched by `zoink.swarm` + `zoink.agent` labels
- **Schema validation with line numbers** — `zoink config validate` reports issues with file, line, and column context
- **Graceful config reload** — Hot-applies safe changes (`model`, `temperature`, `skills`) and prompts before restarts for critical changes
- **Starter templates** — `default`, `hybrid`, and `coding-team` via `zoink config init --template`

### Runtime & state

- **Persistent agent state** — Workspace bind mounts under `workspace/agent-state/<agent>/<runtime>/` survive `zoink down` / `zoink up`. Disable with `persistState: false`. Export with `zoink agent state export`.
- **Health-aware orchestration** — Framework-specific Docker health checks drive `zoink ps`, dashboards, and crash-loop detection
- **Crash-loop visibility** — Repeated unhealthy containers surface in the activity feed with auto-recovery after 30 s of healthy checks
- **Resource governance** — `cpu_limit` / `memory_limit` become Docker constraints; `zoink agent stats` shows live usage
- **Dynamic agent management** — Spawn, remove, pause, unpause, and exec into agents at runtime
- **Sidecar config files** — `configFiles: [{source, target, mode}]` drops verbatim files into an agent's state mount before boot
- **SOUL.md seeding** — Per-agent tone, style, personality, and rules auto-seeded into the container

### Coordination

- **Built-in Kanban board** — Shared `kanban.yaml` with `add`, `move`, `assign`, and `watch` commands
- **PM-driven delegation** — `zoink run task "<goal>"` sends a high-level goal to the PM agent for decomposition
- **Direct agent queries** — Stream a prompt to any agent role with `zoink run query --agent <role>`
- **Autonomous task pickup** — Workers with `autoPickup: true` self-claim assigned `To Do` cards (with `repoUrl`), clone the repo into their persistent state, run, and move the card to `Done`
- **Human-in-the-loop approval** — Configurable approval flow with timeout, caching, and audit history

### Interfaces

- **Terminal dashboard** — Five tabs (Overview, Kanban, Logs, Agents, Chat) with approval modal
- **Web dashboard** — Browser SPA mirroring the TUI, served by `zoink daemon` with login + cookie auth
- **Chat with agents** — Stream conversations against any gateway-enabled agent via OpenResponses, with a `chat_completions` fallback for legacy gateways
- **Daemon mode** — `zoink daemon start` runs a long-lived HTTP service for remote Kanban, agent control, chat, and SSE events
- **Remote Kanban** — `kanban: { mode: remote }` routes every Kanban command and pickup loop through the daemon over HTTP
- **Log streaming** — Color-coded multi-agent logs with `--follow`, `--tail`, `--since`, `--until`, and automatic API-key redaction

### Configuration & customization

- **Provider/auth block** — `provider.name` + `keyEnv` / `key` / `baseUrl` for OpenAI, Anthropic, OpenRouter, Google, and OpenAI-compatible gateways
- **Global config** — `~/.zoink/config.yml` for `default_model`, `dashboard.theme`, `chat.adapter`, and `api_keys.<provider>`
- **Role templates** — Built-in (`pm`, `coder`, `researcher`, `devops`) plus user-created templates under `~/.zoink/roles/`

---

## Commands

### Swarm lifecycle

| Command | Description |
| --- | --- |
| `zoink up` | Launch the swarm, create the network, initialize `kanban.yaml` |
| `zoink down` | Stop and remove swarm containers and network (state is preserved) |
| `zoink ps` | Show name, role, runtime, status, health, container ID |
| `zoink dashboard` | Open the terminal dashboard |

### Agent management

| Command | Description |
| --- | --- |
| `zoink agent spawn <name>` | Add a new agent using runtime-aware defaults |
| `zoink agent rm <name>` | Remove an agent container |
| `zoink agent pause / unpause <name>` | Pause / resume a container |
| `zoink agent exec <name> [cmd]` | Run a command inside an agent |
| `zoink agent logs <name> [-f] [--tail N]` | Stream logs with filtering and redaction |
| `zoink agent stats <name> [--stream]` | One-shot or live Docker resource stats |
| `zoink agent state export <name>` | Tar an agent's persistent state directory |

### Configuration

| Command | Description |
| --- | --- |
| `zoink config init [--template <name>]` | Generate a v2 `swarm.yaml` (templates: `default`, `hybrid`, `coding-team`) |
| `zoink config validate` | Validate `swarm.yaml` against the v2 schema with line numbers |
| `zoink config reload [--mode hot\|hybrid\|restart\|off]` | Apply safe changes and prompt for restarts on critical changes |
| `zoink config set / get` | Read or write global config in `~/.zoink/config.yml` |
| `zoink config role list / create / delete` | Manage built-in and user-created role templates |

### Kanban board

| Command | Description |
| --- | --- |
| `zoink kanban show` | Display the shared board |
| `zoink kanban add --title <text> [--repo-url <url>] [--assign <agent>]` | Add a card; `--repo-url` enables autonomous pickup |
| `zoink kanban move <id> <column>` | Move between `To Do`, `In Progress`, `Done` |
| `zoink kanban assign <id> <agent>` | Assign a card |
| `zoink kanban watch [--json]` | Watch the board (debounced diffs or JSON events) |

### Task orchestration

| Command | Description |
| --- | --- |
| `zoink run task "<description>" [--repo-url <url>]` | Delegate to the PM agent; falls back to a local card if unreachable |
| `zoink run query "<prompt>" --agent <role>` | Stream a direct query to any agent role |

### Daemon

| Command | Description |
| --- | --- |
| `zoink daemon start --token-env <VAR> [--port 9999] [--bind 127.0.0.1]` | Start the HTTP daemon (web SPA + APIs) |
| `zoink daemon status` | Report bind, workspace, auth source, PID |
| `zoink daemon stop` | Gracefully stop the running daemon |

---

## Dashboards

Both the terminal dashboard (`zoink dashboard`) and the browser dashboard (served by `zoink daemon start`) provide the same five-tab experience:

| Tab | Features |
| --- | --- |
| **Overview** | Swarm health summary, activity feed, crash-loop events, command/task input |
| **Kanban** | Interactive board backed by `kanban.yaml`, with add/move/assign and approval modal |
| **Logs** | Multi-agent log viewer with filtering, follow mode, secret redaction |
| **Agents** | Runtime, health, status, resource stats, pause/exec controls |
| **Chat** | Stream a conversation with any gateway-enabled agent (history persisted to `chat-history.yaml`) |

The TUI honors `zoink config set dashboard.theme dark|light`. The web dashboard adds a login page that exchanges a bearer token for a `zoink_session` HttpOnly cookie.

---

## Daemon & remote Kanban

A single `zoink daemon` instance can serve:

- The web SPA (`GET /`)
- Kanban CRUD + SSE board events (`/kanban`, `/kanban/events`)
- Agent control (`/api/agents/:name/{pause,unpause,restart,logs/events,stats/events}`)
- Chat (`/api/agents/:name/chat/...`)
- Approvals (`/api/approvals/:requestId/decision`)
- Login / cookie auth (`/auth/login`, `/auth/logout`, `/auth/me`)
- Generic event stream (`/events`) and health (`/health`)

All API routes require `Authorization: Bearer <token>` or the `zoink_session` cookie. The daemon binds to `127.0.0.1` by default; expose it wider with `--bind 0.0.0.0` and put TLS termination in front of it (nginx, Caddy, etc.).

To share one board across multiple swarm hosts, set `kanban: { mode: remote, daemonUrl, authTokenEnv }` in each consumer `swarm.yaml`. Every `zoink kanban` subcommand and the autonomous-pickup cron will then talk to the daemon over HTTP instead of touching `kanban.yaml` directly.

---

## Runtimes & roles

### Runtimes

| Runtime | Default image | What it adds |
| --- | --- | --- |
| `openclaw` (default) | `ghcr.io/openclaw/openclaw:latest` | OpenClaw defaults, process-based health check, config mount at `/root/.openclaw/config` |
| `hermes` | `nousresearch/hermes-agent:v2026.4.23` | Hermes defaults, profile support, HTTP `/health` check, data mount at `/opt/data` |

Mixed-framework swarms share one Docker network and one workspace, so OpenClaw and Hermes agents collaborate inside the same project.

### Built-in roles

- **pm** — Project manager; decomposes goals and assigns work
- **coder** — Writes code, implements features, fixes bugs
- **researcher** — Gathers information, evaluates libraries, reads docs
- **devops** — Manages infrastructure, deployment, CI/CD

---

## Requirements

- **Node.js** `>= 20`
- **Docker Engine 24+** (or Docker Desktop) running locally
- An **LLM API key** for your provider of choice (OpenAI, Anthropic, OpenRouter, Google, or any OpenAI-compatible gateway)

---

## License & credits

Zoink is released under the [MIT License](https://github.com/zoink-dev/zoink-cli/blob/main/LICENSE).

**Made with care by the Zoink community.**
