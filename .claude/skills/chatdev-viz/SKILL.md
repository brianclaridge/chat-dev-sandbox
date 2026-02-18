---
name: chatdev-viz
description: Generate data visualizations from a file using ChatDev. Use when the user asks to visualize, chart, plot, or analyze data from a CSV, Excel, or other data file.
argument-hint: [description of visualization] [path/to/data.csv]
allowed-tools: Bash, Read, Write, Glob
---

# ChatDev Viz — Data Visualization

Generate charts and visualizations from data files using the ChatDev data visualization workflow.

## Instructions

You are orchestrating a ChatDev workflow via its REST API at `http://localhost:6400`.

Given the user's request: **$ARGUMENTS**

Follow these steps:

### 1. Identify the data file

Parse `$ARGUMENTS` to find the file path. If the user provided a path, verify it exists.
If no file was specified, ask the user for the data file path.

### 2. Upload the data file

```bash
SESSION_ID="viz-$(date +%s)"
curl -s -X POST "http://localhost:6400/api/uploads/$SESSION_ID" \
  -F "file=@/path/to/data.csv"
```

Save the `attachment_id` from the response.

### 3. Start the visualization workflow

```bash
curl -s -X POST http://localhost:6400/api/workflow/execute \
  -H 'Content-Type: application/json' \
  -d "{\"yaml_file\":\"data_visualization_enhanced_v3.yaml\",\"task_prompt\":\"$ARGUMENTS\",\"session_id\":\"$SESSION_ID\",\"attachments\":[\"ATTACHMENT_ID\"]}"
```

### 4. Poll for artifacts

Loop with long-polling, filtering for image outputs:

```bash
CURSOR=0
curl -s "http://localhost:6400/api/sessions/$SESSION_ID/artifact-events?wait_seconds=25&after=$CURSOR&include_mime=image/"
```

Update `CURSOR` to `next_cursor`. Stop when `timed_out` is true with no events for 2
consecutive polls.

### 5. Download generated charts

For each image artifact returned, download it:

```bash
mkdir -p ./output/$SESSION_ID
curl -s "http://localhost:6400/api/sessions/$SESSION_ID/artifacts/ARTIFACT_ID?mode=stream" \
  -o "./output/$SESSION_ID/FILENAME"
```

### 6. Show results

List all downloaded visualizations and tell the user where to find them.
Read the output directory and describe what charts were generated.
