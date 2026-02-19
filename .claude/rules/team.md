# /chatdev Team Architecture

## Overview

`/chatdev [prompt]` invokes a single skill (`.claude/skills/chatdev/SKILL.md`) that
spawns a 4-agent team using Claude Code's experimental agent teams feature
(`CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` in settings).

```
User: /chatdev "A space shooter with power-ups"
  │
  ▼
CTO (lead session, from SKILL.md)
  ├── architect  → designs workflow YAML graph
  ├── engineer   → executes workflow via API, downloads artifacts
  └── reviewer   → QAs output, produces summary
```

## Agent Roles

| Agent | Agent File | Tools | Responsibility |
|---|---|---|---|
| CTO | SKILL.md (inline) | TeamCreate, Task, TaskCreate, TaskUpdate, SendMessage | Coordination only — never implements |
| Architect | `.claude/agents/architect/AGENT.md` | Bash, Read, Write, Glob, Grep | Design + upload workflow YAML |
| Engineer | `.claude/agents/engineer/AGENT.md` | Bash, Read, Write, Glob | Execute workflow, poll artifacts, download results |
| Reviewer | `.claude/agents/reviewer/AGENT.md` | Bash, Read, Glob, Grep | Validate output, summarize for user |

## Task Flow

1. CTO creates team `chatdev-{timestamp}` via `TeamCreate`
2. CTO creates 3 tasks via `TaskCreate` with dependencies: #1 → #2 → #3
3. CTO spawns 3 teammates via `Task` tool (all `subagent_type: "general-purpose"`)
4. CTO assigns tasks via `TaskUpdate` (owner field)
5. Teammates work sequentially (blocked by dependencies):
   - Architect uploads YAML → messages engineer with filename
   - Engineer executes workflow → messages reviewer with output path
   - Reviewer validates output → messages CTO with summary
6. CTO presents results to user, then shuts down team

## Workflow YAML Design

The architect creates workflows from scratch every time. Key points:

- **No templates or sample workflows** — `task clean-workflows` removes all samples on startup
- **Schema**: directed graph of nodes (agent, python, compose, human) + edges with JS conditions
- **Env vars**: `${MODEL_NAME}`, `${BASE_URL}`, `${API_KEY}` — resolved at runtime from `.env`
- **Upload**: `POST /api/workflows/upload/content` with `{"filename":"...","content":"..."}`
- **Validate**: `POST /api/config/schema/validate` with `{"document":"...","breadcrumbs":null}`

## ChatDev Backend API (used by engineer)

```
POST /api/workflow/execute        → start workflow (yaml_file, task_prompt, session_id)
GET  /api/sessions/{id}/artifact-events?wait_seconds=30&after={cursor}  → long-poll artifacts
GET  /api/sessions/{id}/download  → download session ZIP
POST /api/uploads/{id}            → upload attachment (multipart)
```

## Output

Generated files land in `.data/output/{session_id}/`. The reviewer reads these
and produces a structured summary (files, what was built, how to run, quality assessment).
