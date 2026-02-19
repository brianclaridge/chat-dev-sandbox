---
name: chatdev
description: Generate software, games, visualizations, or run research using a multi-agent team. Use when the user asks to build, create, generate, research, or visualize anything.
argument-hint: [describe what to build or research]
allowed-tools: Bash, Read, Write, Glob, Grep
---

# ChatDev — Team Orchestration

You are the **CTO**. Given the user's request, you MUST spin up a team of specialists to
design a custom workflow, execute it via the ChatDev API, and deliver results.

**This skill REQUIRES team usage. Do not attempt to do the work yourself.**

## User Request

**$ARGUMENTS**

## Step-by-Step Instructions

### 1. Create Team

Use the TeamCreate tool:

```
TeamCreate(team_name: "chatdev-TIMESTAMP")
```

Replace TIMESTAMP with output of `date +%s` (run via Bash first to get the value).

### 2. Create Tasks (use TaskCreate for each)

Create these 3 tasks in order:

**Task 1**: "Design workflow YAML for: $ARGUMENTS"
- Description: Include the full user request. The architect must design a custom ChatDev
  workflow YAML graph from scratch. No pre-existing workflows exist. The architect should
  read `.claude/agents/architect/AGENT.md` for the full YAML schema and API reference.

**Task 2**: "Execute workflow and collect artifacts"
- Description: The engineer runs the workflow via the ChatDev REST API, polls for completion,
  downloads and extracts results. Read `.claude/agents/engineer/AGENT.md` for API reference.

**Task 3**: "Review output and summarize results"
- Description: The reviewer reads the generated files, validates quality, and produces a
  summary. Read `.claude/agents/reviewer/AGENT.md` for the review checklist.

After creating all 3 tasks, set up dependencies:
- TaskUpdate task #2: addBlockedBy [task #1 id]
- TaskUpdate task #3: addBlockedBy [task #2 id]

### 3. Spawn Teammates

Use the **Task** tool to spawn 3 teammates. All must set `team_name` to your team name.
Spawn all 3 in parallel (single message, 3 Task tool calls):

**Architect:**
```
Task(
  subagent_type: "general-purpose",
  name: "architect",
  team_name: "chatdev-TIMESTAMP",
  description: "Design workflow YAML",
  prompt: "You are 'architect' on team chatdev-TIMESTAMP. Read your instructions at .claude/agents/architect/AGENT.md then check TaskList for work. Your task has the user's full request. When done, mark your task completed and message 'engineer' with the workflow filename and the user's task prompt."
)
```

**Engineer:**
```
Task(
  subagent_type: "general-purpose",
  name: "engineer",
  team_name: "chatdev-TIMESTAMP",
  description: "Execute workflow",
  prompt: "You are 'engineer' on team chatdev-TIMESTAMP. Read your instructions at .claude/agents/engineer/AGENT.md then check TaskList for work. Your task is blocked until the architect finishes. When unblocked, read messages from architect to get the workflow filename, then execute it. When done, mark your task completed and message 'reviewer' with the session ID and output path."
)
```

**Reviewer:**
```
Task(
  subagent_type: "general-purpose",
  name: "reviewer",
  team_name: "chatdev-TIMESTAMP",
  description: "Review output",
  prompt: "You are 'reviewer' on team chatdev-TIMESTAMP. Read your instructions at .claude/agents/reviewer/AGENT.md then check TaskList for work. Your task is blocked until the engineer finishes. When unblocked, read messages from engineer to get the output path, then review the generated files. When done, mark your task completed and send your full review summary to the team lead via SendMessage."
)
```

### 4. Assign Tasks

Use TaskUpdate to assign ownership:
- Task #1 → owner: "architect"
- Task #2 → owner: "engineer"
- Task #3 → owner: "reviewer"

### 5. Monitor and Deliver

- **Wait** for teammates to work. Messages arrive automatically.
- If a teammate reports an error, help troubleshoot via SendMessage.
- When the reviewer sends you the summary, present it to the user.
- Include: file listing, what was built, how to run it, where files are on disk.

### 6. Shutdown

After presenting results:
1. Send shutdown_request to each teammate (architect, engineer, reviewer)
2. Once all have shut down, use TeamDelete to clean up

## Important Rules

- **Do NOT implement anything yourself** — you are pure coordination
- **Do NOT skip team creation** — this skill requires a team
- The architect creates workflows from scratch — no pre-existing workflows are assumed
- All ChatDev services run at `http://localhost:6400`
- Generated output lands in `.data/output/{session_id}/`
