---
name: chatdev-validate
description: Validate a ChatDev workflow YAML file against the configuration schema. Use when the user asks to check, lint, or validate a workflow file.
argument-hint: [path/to/workflow.yaml]
allowed-tools: Bash, Read, Glob
---

# ChatDev Validate — Workflow YAML Linter

Validate a ChatDev workflow YAML file against the official schema.

## Instructions

You are validating a workflow YAML using the ChatDev API at `http://localhost:6400`.

Given the user's input: **$ARGUMENTS**

Follow these steps:

### 1. Locate the file

If `$ARGUMENTS` is a file path, read it. If it's a workflow name (e.g., `ChatDev_v1`),
fetch it from the API:

```bash
# From API
curl -s "http://localhost:6400/api/workflows/ChatDev_v1.yaml" | jq -r .content

# Or read local file
cat /path/to/workflow.yaml
```

### 2. Validate against the schema

```bash
CONTENT=$(cat /path/to/workflow.yaml | jq -Rs .)
curl -s -X POST http://localhost:6400/api/config/schema/validate \
  -H 'Content-Type: application/json' \
  -d "{\"document\":$CONTENT,\"breadcrumbs\":null}"
```

### 3. Report results

If the response contains `"valid": true`:
- Tell the user the workflow is valid

If the response contains `"valid": false`:
- Show the `error` message
- Show the `path` to the invalid field
- Suggest how to fix the issue based on the schema
- If possible, read the relevant part of the YAML and propose a corrected version
