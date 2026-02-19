# ChatDev Docker Sandbox

Local multi-agent AI workflow platform. Configurable LLM backend (LM Studio, OpenAI, etc.).

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

## LLM Configuration

Set your provider in `.env`:
```env
BASE_URL=http://host.docker.internal:1234/v1   # LM Studio on host
API_KEY=lm-studio
MODEL_NAME=qwen3-coder-next
```

The backend container reaches the host via `host.docker.internal`.

## Commands

```text
task up           Start all services
task down         Stop all services
task restart      Clean slate rebuild
task ps           Show status
task logs         Tail logs
task shell        Shell into backend
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
