# ChatDev Docker Sandbox

## Quick Start

```bash
task setup        # create .env from template (one-time)
task up           # build, start all services, pull default model
```

Open http://localhost:5173 to use ChatDev.

## Current State (2026-02-18)

All services operational. Default model: `qwen3-coder-next` (51GB).

| Service | Container | Status | URL |
|---|---|---|---|
| Frontend (Vite + Vue 3) | chatdev_frontend | running | http://localhost:5173 |
| Backend (FastAPI) | chatdev_backend | healthy | http://localhost:6400 |
| Ollama (LLM serving) | chatdev_ollama | healthy | http://localhost:11435 |

Models loaded:
- `qwen3-coder-next:latest` (51 GB) -- default
- `qwen2.5-coder:7b` (4.7 GB) -- previous default, still cached

---

## Task Commands

```bash
# --- Lifecycle ---
task setup              # create .env from template (one-time)
task up                 # build + start + pull default model
task down               # stop all containers
task restart            # stop + rebuild + start
task ps                 # show running services

# --- Logs ---
task logs               # tail all services
task logs:backend       # tail backend only
task logs:ollama        # tail ollama only

# --- Debug ---
task shell              # bash into backend container

# --- Ollama Models ---
task ollama:list        # list pulled models
task ollama:pull        # pull default model (qwen3-coder-next)
task ollama:pull-custom -- <model>   # pull any model by name

# --- Maintenance ---
task validate           # validate YAML workflow files
task sync               # sync Vue graphs to DB
task clean              # stop + wipe all .data/ (with confirmation)
```

---

## Claude Code Skills

Slash commands that integrate with the ChatDev API. Services must be running (`task up`).

| Command | What it does |
|---|---|
| `/chatdev [prompt]` | Generate a full software project from a description |
| `/chatdev-game [prompt]` | Generate a pygame game |
| `/chatdev-viz [prompt] [file]` | Generate charts/visualizations from a data file |
| `/chatdev-research [topic]` | Run multi-agent deep research on a topic |
| `/chatdev-validate [file]` | Validate a workflow YAML against the schema |
| `/chatdev-workflows [name?]` | List all workflows, or inspect a specific one |
| `/chatdev-status [session-id]` | Check progress/results of a running workflow |

```
.claude/
├── agents/
│   └── skill-guru/AGENT.md        <- agent for creating new skills
└── skills/
    ├── chatdev/SKILL.md           <- software project generation
    ├── chatdev-game/SKILL.md      <- game generation
    ├── chatdev-viz/SKILL.md       <- data visualization
    ├── chatdev-research/SKILL.md  <- deep research
    ├── chatdev-validate/SKILL.md  <- workflow YAML validation
    ├── chatdev-workflows/SKILL.md <- list/inspect workflows
    └── chatdev-status/SKILL.md    <- check session status
```

All skills use the same async pattern: `POST /api/workflow/execute` -> long-poll
`/api/sessions/{id}/artifact-events` -> download results. The `skill-guru` agent
knows the full ChatDev API and skill authoring spec for creating new skills.

---

## Architecture

```
openbmb/
├── CLAUDE.md                  <- this file
├── Taskfile.yml               <- task runner (project root)
├── .env                       <- user config, created by task setup (gitignored)
├── .setup/
│   ├── compose.yml            <- Docker Compose (backend, frontend, ollama)
│   ├── .env.example           <- template copied to ../.env
│   └── .env.docker            <- Docker-internal overrides (bind addr, CORS)
├── .data/
│   ├── ollama/                <- model weights (~55GB with current models)
│   ├── warehouse/             <- ChatDev generated projects
│   ├── logs/                  <- server + workflow logs
│   └── data/                  <- vuegraphs.db and app data
└── ChatDev/                   <- UNTOUCHED upstream repo
```

### Service Diagram

```
┌──────────────────────────────────────────────────────────────┐
│  .setup/compose.yml                                           │
│                                                                │
│  ┌───────────┐   ┌───────────┐   ┌────────────────────────┐  │
│  │ frontend  │──>│  backend  │──>│  ollama                │  │
│  │ :5173     │   │  :6400    │   │  :11435 (host)         │  │
│  │ (vite)    │   │  (fastapi)│   │  :11434 (container)    │  │
│  └───────────┘   └───────────┘   └────────────────────────┘  │
│       │               │                    │                   │
│       │proxy          │                    │                   │
│       │/api,/ws ──────┘               .data/ollama/           │
│                  .data/warehouse/                              │
│                  .data/logs/                                   │
│                  .data/data/                                   │
└──────────────────────────────────────────────────────────────┘
```

### Networking

| From | To | URL |
|---|---|---|
| Browser | Frontend (Vite) | `http://localhost:5173` |
| Vite proxy (`/api`, `/ws`) | Backend | `http://backend:6400` (Docker internal) |
| Backend | Ollama | `http://ollama:11434/v1` (Docker internal) |
| Backend | External APIs | Outbound HTTPS (only if configured in `.env`) |

The frontend uses **relative URLs** (`/api/...`). Vite's dev server proxies these
to the backend container. The browser never talks to port 6400 directly.

---

## Constraint

**No files inside `./ChatDev/` are created or modified.** We reuse ChatDev's existing
Dockerfiles by pointing `build.context` at `../ChatDev` from the compose file.

---

## Configuration

### .env (user-facing, at project root)

```env
# Default: containerized Ollama
BASE_URL=http://ollama:11434/v1
API_KEY=ollama

# Alternative: host Ollama
# BASE_URL=http://host.docker.internal:11434/v1

# Alternative: OpenAI
# BASE_URL=https://api.openai.com/v1
# API_KEY=sk-...

# Ollama container port on host (default 11435 to avoid conflict with host Ollama)
OLLAMA_HOST_PORT=11435
```

### Key design details

- **Taskfile.yml** uses a `DC` variable for all docker compose commands:
  `docker compose -f .setup/compose.yml --env-file .env`
- **Volumes are bind mounts** to `.data/`, not named Docker volumes. Data is
  directly visible on the host filesystem.
- **Ollama host port defaults to 11435** to avoid conflict if Ollama is already
  running on the host at 11434.
- **Ollama memory limit is 64GB** to accommodate large models like qwen3-coder-next.
- **Backend has resource limits**: 2 CPUs, 4GB RAM, 256 PIDs.

---

## Issues Found & Fixed During Setup

1. **GPU driver error**: `could not select device driver "default" with capabilities: [[gpu]]`
   - Removed `deploy.resources.reservations.devices` from ollama service.
     GPU passthrough requires nvidia-container-toolkit on the host.

2. **Ollama healthcheck fails**: `curl: not found` in ollama/ollama image.
   - Changed healthcheck to `["CMD", "ollama", "list"]`.

3. **Port 11434 conflict**: Host Ollama already bound to 11434.
   - Default host port changed to 11435 via `${OLLAMA_HOST_PORT:-11435}:11434`.

4. **Frontend 500 errors on /api/workflows and /api/config/schema**:
   - Root cause: `VITE_API_BASE_URL` was `http://localhost:6400`. The Vite dev server
     (inside the frontend container) uses this as the proxy target. Inside the container,
     `localhost` is the container itself, not the backend.
   - Fix: Changed to `http://backend:6400` (Docker service name). The browser never
     sees this value -- the frontend uses relative URLs and Vite proxies them.
