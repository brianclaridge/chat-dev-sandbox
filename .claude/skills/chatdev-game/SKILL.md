---
name: chatdev-game
description: Generate a pygame game using ChatDev GameDev workflow. Use when the user asks to create, build, or make a game.
argument-hint: [description of the game to build]
allowed-tools: Bash, Read, Write, Glob
---

# ChatDev Game — Generate a Pygame Game

Generate a game by running the ChatDev GameDev multi-agent workflow.

## Instructions

You are orchestrating a ChatDev workflow via its REST API at `http://localhost:6400`.

Given the user's request: **$ARGUMENTS**

Follow these steps:

### 1. Start the GameDev workflow

```bash
SESSION_ID="game-$(date +%s)"
curl -s -X POST http://localhost:6400/api/workflow/execute \
  -H 'Content-Type: application/json' \
  -d "{\"yaml_file\":\"GameDev_v1.yaml\",\"task_prompt\":\"$ARGUMENTS\",\"session_id\":\"$SESSION_ID\"}"
```

Report the session ID to the user.

### 2. Poll for artifacts

Loop with long-polling until the workflow completes:

```bash
CURSOR=0
curl -s "http://localhost:6400/api/sessions/$SESSION_ID/artifact-events?wait_seconds=25&after=$CURSOR"
```

Between polls, tell the user what's happening. Update `CURSOR` to `next_cursor`.
Stop when `timed_out` is true with no events for 2 consecutive polls.

### 3. Download and extract results

```bash
mkdir -p ./output/$SESSION_ID
curl -s "http://localhost:6400/api/sessions/$SESSION_ID/download" -o "./output/$SESSION_ID/session.zip"
cd ./output/$SESSION_ID && unzip -o session.zip
```

### 4. Show the user what was generated

List the files, read the main game script, and summarize:
- Game mechanics and controls
- How to run it (e.g., `python main.py`)
- What pygame features were used
