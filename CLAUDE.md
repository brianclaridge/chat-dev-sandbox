# ChatDev Docker Sandbox

Self-contained multi-agent software generation platform. Users describe what to build;
a graph of LLM agents (analyst, coder, reviewer, etc.) collaborates to produce working
code. Runs entirely offline via Ollama.

## Stack

| Layer | Tech | Container | Port |
|---|---|---|---|
| Frontend | Vite + Vue 3 | chatdev_frontend | 5173 |
| Backend | FastAPI (Python) | chatdev_backend | 6400 |
| LLM | Ollama | chatdev_ollama | 11435 (host) / 11434 (internal) |

Default model: `glm-4.7-flash` (~19 GB). Set via `MODEL_NAME` in `.env`.

## Commands

```bash
task setup        # create .env from template (one-time)
task up           # clone + build + start + pull model + clean sample workflows
task down         # stop containers
task restart      # clean slate: stop + re-clone + rebuild + start
task ps           # show status
task logs         # tail all services
task shell        # bash into backend
task clean        # wipe all .data/ (destructive)
```

## Constraint

`.src/` is **ephemeral** — auto-cloned from upstream ChatDev repo, destroyed on
`task restart`. Never commit changes there. All customization lives in `.setup/`,
`.env`, `Taskfile.yml`, and `.claude/`.

## Skill

One entry point: `/chatdev [prompt]`. Spawns a CTO-led team (architect → engineer →
reviewer) that designs a custom workflow YAML from scratch, executes it via the backend
API, and delivers results. No pre-existing workflows assumed.

See @.claude/rules/services.md for networking and configuration details.
See @.claude/rules/team.md for team architecture and agent roles.
