# ChatDev Docker Sandbox

Local multi-agent AI workflow platform. Runs entirely offline with Ollama.

## Prerequisites

- [Docker](https://docs.docker.com/get-docker/)
- [Task](https://taskfile.dev/installation/) (`go-task`)

## Quickstart

```bash
git clone https://github.com/OpenBMB/ChatDev.git
task setup
task up
```

Open http://localhost:5173

## Claude Code Skills

Requires services running (`task up`).

```bash
/chatdev Build a CLI tool that converts CSV to JSON
/chatdev-game A space shooter with power-ups
/chatdev-viz Analyze trends from ./data/sales.csv
/chatdev-research Compare Rust vs Go for CLI tools
/chatdev-validate ./my-workflow.yaml
/chatdev-workflows
/chatdev-status <session-id>
```

## Commands

```text
task up           Start all services
task down         Stop all services
task ps           Show status
task logs         Tail logs
task shell        Shell into backend

task ollama:list                   List models
task ollama:pull-custom -- <name>  Pull a model

task clean        Wipe all data
```

## Layout

```text
.setup/           Docker Compose + env files
.data/            Persistent data (models, outputs, logs)
.claude/skills/   Claude Code slash commands
ChatDev/          Upstream repo (untouched)
```
