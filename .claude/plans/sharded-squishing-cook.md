# Plan: Run `/chatdev A space shooter with power-ups`

## Context

All infrastructure is in place. This is an execution task — invoke the `/chatdev` skill
and verify the team system works end-to-end with Ollama.

## Pre-flight Verification (DONE)

- [x] Backend running at `http://localhost:6400` — `{"workflows":[]}`
- [x] Ollama running with `qwen3-coder-next` (51.7 GB) at `:11435`
- [x] `.env` configured for Ollama: `BASE_URL=http://ollama:11434/v1`, `MODEL_NAME=qwen3-coder-next`
- [x] `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1` enabled in settings
- [x] All sample workflows cleaned (0 remaining)
- [x] Agent files: architect, engineer, reviewer — all present
- [x] Old skill-guru agent deleted, only 3 correct agents exist

## Execution Steps

### 1. Invoke `/chatdev A space shooter with power-ups`

Use the Skill tool to invoke the `chatdev` skill with args `A space shooter with power-ups`.

### 2. Follow SKILL.md instructions (CTO role)

The skill will instruct me to:
1. `TeamCreate("chatdev-{timestamp}")`
2. `TaskCreate` × 3 (design, execute, review) with dependencies
3. `Task` × 3 to spawn architect, engineer, reviewer (all `general-purpose`, with `team_name`)
4. `TaskUpdate` to assign ownership
5. Monitor via incoming messages
6. Present results to user
7. Shutdown team + `TeamDelete`

### 3. Verify Ollama Usage

The architect's workflow YAML must use `${MODEL_NAME}` / `${BASE_URL}` / `${API_KEY}` —
these resolve to `qwen3-coder-next` / `http://ollama:11434/v1` / `ollama` at runtime.

### 4. Verify Team Usage

Confirm all 3 teammates spawn, communicate via SendMessage, and tasks flow through
the dependency chain: architect → engineer → reviewer.

## Expected Output

- Workflow YAML created and uploaded by architect
- Workflow executed by engineer, artifacts downloaded to `.data/output/{session_id}/`
- Review summary from reviewer with file listing, quality assessment, run instructions
- Team shutdown and cleanup

## Commit

After successful execution, commit any generated output if requested.
