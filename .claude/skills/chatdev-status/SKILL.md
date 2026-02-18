---
name: chatdev-status
description: Check the status of a running or completed ChatDev workflow session. Use when the user asks about a workflow's progress, wants to see results, or check if a session finished.
argument-hint: [session-id]
allowed-tools: Bash, Read, Glob
---

# ChatDev Status — Check Workflow Session

Check the status and results of a ChatDev workflow session.

## Instructions

You are querying the ChatDev API at `http://localhost:6400`.

Given the user's input: **$ARGUMENTS**

### 1. Find the session

If `$ARGUMENTS` is a session ID, use it directly.

If no session ID provided, look for recent sessions:

```bash
# List recent output directories
ls -lt ./output/ 2>/dev/null | head -10
```

### 2. Check for artifacts

```bash
SESSION_ID="$ARGUMENTS"
curl -s "http://localhost:6400/api/sessions/$SESSION_ID/artifact-events?wait_seconds=1&after=0"
```

### 3. Report status

Based on the response, tell the user:

**If events exist:**
- Number of artifacts produced
- File names, types, and sizes
- Whether the workflow appears complete (`has_more: false` and `timed_out: true`)

**If no events:**
- The workflow may still be running (suggest waiting and checking again)
- Or the session ID may be invalid

### 4. Offer next steps

- To download results: `curl -s "http://localhost:6400/api/sessions/SESSION_ID/download" -o session.zip`
- To view a specific artifact: fetch it by artifact_id
- If the workflow is still running, offer to poll and wait

### 5. If output directory exists locally

```bash
ls -la ./output/$SESSION_ID/ 2>/dev/null
```

Show what's already been downloaded and summarize the contents.
