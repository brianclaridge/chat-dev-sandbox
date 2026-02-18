---
name: chatdev-workflows
description: List, inspect, or describe available ChatDev workflows. Use when the user asks what workflows exist, wants to see a workflow's configuration, or asks about available ChatDev capabilities.
argument-hint: [workflow-name (optional)]
allowed-tools: Bash, Read
---

# ChatDev Workflows — List and Inspect

Browse available ChatDev workflows and inspect their configuration.

## Instructions

You are querying the ChatDev API at `http://localhost:6400`.

Given the user's input: **$ARGUMENTS**

### If no arguments (list all workflows)

```bash
curl -s http://localhost:6400/api/workflows | jq .
```

Present the list organized by category:
- **Software Development**: ChatDev_v1, etc.
- **Game Development**: GameDev_v1, etc.
- **Data Visualization**: data_visualization_*, etc.
- **Research**: deep_research_*, etc.
- **Demos**: demo_*, etc.
- **Other**: everything else

Add a brief description of what each category does.

### If a workflow name is provided (inspect it)

```bash
# Add .yaml suffix if needed
curl -s "http://localhost:6400/api/workflows/$ARGUMENTS.yaml" | jq -r .content
```

If that 404s, try without adding .yaml:
```bash
curl -s "http://localhost:6400/api/workflows/$ARGUMENTS" | jq -r .content
```

Parse the YAML content and present:
- **Description**: from the `graph.description` field
- **Node count**: how many agents/steps
- **Node types**: what kinds of nodes (agent, human, python, etc.)
- **Agent roles**: names and roles of AI agents in the workflow
- **Flow**: start node -> edges -> how the workflow progresses
- **Required variables**: any `${VAR}` references that need `.env` configuration
