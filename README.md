# ChatDev Docker Sandbox

Local multi-agent AI workflow platform. Runs entirely offline with Ollama.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Task](https://taskfile.dev/installation/) (`go-task`)

## Quickstart

```bash
task setup
task up
```

Open http://localhost:5173

## Claude Code

One command to build anything — software, games, visualizations, research:

```bash
/chatdev A space shooter with power-ups
/chatdev Build a CLI tool that converts CSV to JSON
/chatdev Analyze trends from ./data/sales.csv
/chatdev Compare Rust vs Go for CLI tools
```

A CTO-led team (architect, engineer, reviewer) spins up, designs a custom
workflow from scratch, executes it, and delivers results.

## Recommended Model

**`glm-4.7-flash`** (Q4_K_M, ~19 GB) — strongest model in the 30B class (Feb 2026).
Won agentic coding benchmarks, 198K context window, dense architecture.

```bash
task ollama:pull-custom -- glm-4.7-flash
```

Set in `.env`:
```env
MODEL_NAME=glm-4.7-flash
```

Requires NVIDIA GPU passthrough in Docker (see `.setup/compose.yml`).

## Commands

```text
task up           Start all services
task down         Stop all services
task restart      Clean slate rebuild
task ps           Show status
task logs         Tail logs
task shell        Shell into backend

task ollama:list                   List models
task ollama:pull-custom -- <name>  Pull a model

task clean-workflows  Remove all sample workflows
task clean            Wipe all data
```

## Layout

```text
.setup/              Docker Compose + env files
.data/               Persistent data (models, outputs, logs)
.claude/skills/      Single /chatdev skill (CTO entry point)
.claude/agents/      architect, engineer, reviewer agents
.src/                Upstream ChatDev repo (auto-cloned, ephemeral)
```
