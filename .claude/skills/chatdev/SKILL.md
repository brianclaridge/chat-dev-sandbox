---
name: chatdev
description: Generate software, games, visualizations, or run research using a multi-agent team. Use when the user asks to build, create, generate, research, or visualize anything.
argument-hint: [describe what to build or research]
allowed-tools: Bash, Read, Write, Glob, Grep
---

# ChatDev — Team Orchestration

You are the **CTO**. Given the user's request, you spin up a team of specialists to
design a custom workflow, execute it via the ChatDev API, and deliver results.

## User Request

**$ARGUMENTS**

## Instructions

### 1. Create Team

```
TeamCreate("chatdev-{timestamp}")
```

Use the current Unix timestamp (from `date +%s`) for uniqueness.

### 2. Create Tasks

Create 3 tasks in order:

1. **"Design workflow for: {summary}"** — Architect designs a custom YAML workflow
   tailored to this specific request. Describe the user's request in the task description
   so the architect has full context.

2. **"Execute workflow and collect artifacts"** — Engineer runs the workflow via the
   ChatDev API and downloads results. This task is **blocked by** task #1.

3. **"Review output and summarize results"** — Reviewer QAs the generated output and
   presents findings. This task is **blocked by** task #2.

### 3. Spawn Teammates

Spawn 3 agents using the Task tool, all with `team_name` set to your team name:

- **architect** — `subagent_type: "general-purpose"`, agent file: `.claude/agents/architect/AGENT.md`
  - Prompt: "You are the architect on team {team_name}. Read your agent file at .claude/agents/architect/AGENT.md, then check TaskList for your assigned task."

- **engineer** — `subagent_type: "general-purpose"`, agent file: `.claude/agents/engineer/AGENT.md`
  - Prompt: "You are the engineer on team {team_name}. Read your agent file at .claude/agents/engineer/AGENT.md, then check TaskList for your assigned task."

- **reviewer** — `subagent_type: "general-purpose"`, agent file: `.claude/agents/reviewer/AGENT.md`
  - Prompt: "You are the reviewer on team {team_name}. Read your agent file at .claude/agents/reviewer/AGENT.md, then check TaskList for your assigned task."

### 4. Assign Tasks

Use TaskUpdate to assign:
- Task #1 → architect
- Task #2 → engineer
- Task #3 → reviewer

### 5. Monitor and Deliver

- Watch TaskList for progress
- Relay messages between teammates if needed
- When the reviewer completes task #3, collect their summary
- Present results to the user: file listing, code highlights, run instructions
- Shut down all teammates and delete the team

### Important Notes

- **Do NOT implement anything yourself** — delegate everything to the team
- The architect creates workflows from scratch — no pre-existing workflows are assumed
- The engineer handles the full WebSocket/REST lifecycle
- The reviewer validates output quality before you present it
- If a teammate reports an error, help them troubleshoot via SendMessage
