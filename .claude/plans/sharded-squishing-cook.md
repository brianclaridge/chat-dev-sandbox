# Plan: Teams-First ChatDev Orchestration

## Context

The current skill architecture has 7 single-agent skills that each assume pre-existing workflow YAMLs. This is brittle — workflows hardcode model names, the sample library clutters the workspace, and there's no adaptive design.

The new architecture: **one skill, one team**. The user says `/chatdev [prompt]` and a CTO-led team spins up, designs a custom workflow from scratch, executes it via the ChatDev API, and delivers results. No pre-existing workflows assumed.

## Changes Overview

### 1. Taskfile.yml — replace `patch-models` with `clean-workflows`

Delete all sample workflows on startup via the API. The team creates fresh ones per task.

```yaml
clean-workflows:
  desc: Remove all sample workflows shipped with ChatDev
  cmds:
    - |
      python3 -c "
      import json, urllib.request
      BASE = 'http://localhost:{{.BACKEND_PORT}}'
      with urllib.request.urlopen(f'{BASE}/api/workflows') as r:
          workflows = json.loads(r.read()).get('workflows', [])
      deleted = 0
      for wf in workflows:
          req = urllib.request.Request(f'{BASE}/api/workflows/{wf}', method='DELETE')
          try:
              urllib.request.urlopen(req)
              deleted += 1
          except Exception as e:
              print(f'  ✗ {wf}: {e}')
      print(f'Cleaned {deleted} sample workflow(s).')
      "
```

In `up`, replace `task: patch-models` → `task: clean-workflows`. Remove `patch-models` entirely.

### 2. Delete existing skills and agents

Remove:
```
.claude/skills/chatdev/
.claude/skills/chatdev-game/
.claude/skills/chatdev-viz/
.claude/skills/chatdev-research/
.claude/skills/chatdev-validate/
.claude/skills/chatdev-workflows/
.claude/skills/chatdev-status/
.claude/agents/skill-guru/
```

### 3. Create one unified skill: `/chatdev`

**`.claude/skills/chatdev/SKILL.md`** — entry point that spawns a team.

The skill instructs Claude to:
1. `TeamCreate` a team named `chatdev-{timestamp}`
2. Create tasks from the user's prompt
3. Spawn 3 teammates: **architect**, **engineer**, **reviewer**
4. Act as **CTO** — delegate, monitor, deliver results
5. Shut down team when done

### 4. Create 3 custom agents

All agents get the full ChatDev API reference baked into their AGENT.md so they can work from first principles — no dependency on sample workflows.

#### a) `.claude/agents/architect/AGENT.md`
- **Role**: Design workflow YAML graphs tailored to the task
- **Tools**: `Bash, Read, Write, Glob, Grep`
- **Knowledge**: Full YAML schema (node types, edge conditions, config structure), `${MODEL_NAME}` / `${BASE_URL}` / `${API_KEY}` env vars
- **Output**: Uploads a validated workflow YAML via `POST /api/workflows/upload/content`

#### b) `.claude/agents/engineer/AGENT.md`
- **Role**: Execute workflows and collect results
- **Tools**: `Bash, Read, Write, Glob`
- **Knowledge**: Full REST + WebSocket protocol, session lifecycle (WS connect → execute → poll → download)
- **Key**: Uses a Python script to handle the WebSocket connection (since `websocket-client` is installed), executes workflow, polls artifacts, downloads and extracts to `.data/output/{session_id}/`

#### c) `.claude/agents/reviewer/AGENT.md`
- **Role**: QA the generated output
- **Tools**: `Bash, Read, Glob, Grep`
- **Knowledge**: How to read extracted session artifacts, validate code, run generated scripts
- **Output**: Summary of what was built, file listing, key code walkthrough, run instructions

### 5. Team Workflow (runtime)

```
User: /chatdev "A space shooter with power-ups"
  │
  ▼
CTO (main Claude session, from SKILL.md)
  ├── TeamCreate("chatdev-{ts}")
  ├── TaskCreate: "Design workflow for: A space shooter with power-ups"
  ├── TaskCreate: "Execute workflow and collect artifacts"
  ├── TaskCreate: "Review output and summarize results"  (blocked by #2)
  │
  ├── Spawn: architect (general-purpose, team_name=chatdev-{ts})
  ├── Spawn: engineer  (general-purpose, team_name=chatdev-{ts})
  ├── Spawn: reviewer  (general-purpose, team_name=chatdev-{ts})
  │
  ├── Assign task #1 → architect
  ├── Assign task #2 → engineer  (blocked by #1)
  ├── Assign task #3 → reviewer  (blocked by #2)
  │
  ▼ (monitor TaskList, relay results, shutdown team)
  │
  ▼
User sees: generated files, code summary, run instructions
```

### 6. Update CLAUDE.md

- Remove all references to 7 individual skills
- Document the single `/chatdev` entry point
- Document the team architecture (CTO → architect, engineer, reviewer)
- Document that workflows are created from scratch per task
- Document `task clean-workflows` replacing `task patch-models`
- Update task commands section
- Note: `MODEL_NAME` env var is still used — the architect bakes `${MODEL_NAME}` into workflow YAMLs it creates

### 7. Update .env / .env.example

Keep `MODEL_NAME=qwen3-coder-next` — still needed. The architect references it when designing workflows.

## Files to Create/Modify

| File | Action |
|---|---|
| `Taskfile.yml` | Replace `patch-models` with `clean-workflows`, wire into `up` |
| `.claude/skills/chatdev/SKILL.md` | Rewrite — team-spawning CTO skill |
| `.claude/agents/architect/AGENT.md` | Create — workflow YAML designer |
| `.claude/agents/engineer/AGENT.md` | Create — API/WS executor |
| `.claude/agents/reviewer/AGENT.md` | Create — output QA |
| `CLAUDE.md` | Rewrite skills/architecture sections |
| `.claude/skills/chatdev-game/` | Delete |
| `.claude/skills/chatdev-viz/` | Delete |
| `.claude/skills/chatdev-research/` | Delete |
| `.claude/skills/chatdev-validate/` | Delete |
| `.claude/skills/chatdev-workflows/` | Delete |
| `.claude/skills/chatdev-status/` | Delete |
| `.claude/agents/skill-guru/` | Delete |

## Key Design Decisions

- **Architect designs fresh workflows every time** — no templates, no samples. The YAML schema + examples in its AGENT.md give it enough to create tailored workflows.
- **Engineer owns the full WS lifecycle** — Python script handles connect → execute → poll → download. No curl hacks.
- **Reviewer runs the output** — validates generated code actually works before presenting.
- **CTO doesn't do implementation** — pure coordination via TaskCreate/TaskUpdate/SendMessage.

## Verification

1. `task restart` — should re-clone, build, start, and clean all sample workflows
2. `/chatdev "A space shooter with power-ups"` — team spawns, architect designs workflow, engineer executes, reviewer presents
3. Check `.data/output/{session_id}/` for extracted results
4. Verify no sample workflows remain: `curl http://localhost:6400/api/workflows` → `{"workflows": [<only the one the architect created>]}`
