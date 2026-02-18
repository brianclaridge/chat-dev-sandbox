---
name: chatdev-research
description: Run a deep multi-agent research workflow on a topic using ChatDev. Use when the user asks to research, investigate, or write a report on a topic.
argument-hint: [research question or topic]
allowed-tools: Bash, Read, Write, Glob
---

# ChatDev Research — Deep Multi-Agent Research

Run a deep research workflow with multiple AI agents collaborating to investigate a topic.

## Instructions

You are orchestrating a ChatDev workflow via its REST API at `http://localhost:6400`.

Given the user's research question: **$ARGUMENTS**

Follow these steps:

### 1. Start the deep research workflow

```bash
SESSION_ID="research-$(date +%s)"
curl -s -X POST http://localhost:6400/api/workflow/execute \
  -H 'Content-Type: application/json' \
  -d "{\"yaml_file\":\"deep_research_v1.yaml\",\"task_prompt\":\"$ARGUMENTS\",\"session_id\":\"$SESSION_ID\"}"
```

Report the session ID to the user. Note that deep research can take several minutes.

### 2. Poll for artifacts

Loop with long-polling. Research workflows produce text/document artifacts:

```bash
CURSOR=0
curl -s "http://localhost:6400/api/sessions/$SESSION_ID/artifact-events?wait_seconds=25&after=$CURSOR"
```

Between polls, tell the user the research is in progress. Update `CURSOR` to
`next_cursor`. Stop when `timed_out` is true with no events for 3 consecutive polls
(research takes longer than code generation).

### 3. Download the research output

```bash
mkdir -p ./output/$SESSION_ID
curl -s "http://localhost:6400/api/sessions/$SESSION_ID/download" -o "./output/$SESSION_ID/session.zip"
cd ./output/$SESSION_ID && unzip -o session.zip
```

### 4. Present findings

Read the main output document and present a summary to the user. Include:
- Key findings
- Sources referenced
- Agent reasoning highlights
- Where to find the full report on disk
