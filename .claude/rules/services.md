# Docker Services & Configuration

## Networking

| From → To | URL |
|---|---|
| Browser → Frontend | `http://localhost:5173` |
| Vite proxy (`/api`, `/ws`) → Backend | `http://backend:6400` (Docker internal) |
| Backend → LLM | `$BASE_URL` (configured in `.env`) |

The frontend uses relative URLs (`/api/...`). Vite proxies them to the backend
container. The browser never talks to port 6400 directly.

## .env

```env
BASE_URL=http://host.docker.internal:1234/v1   # LLM endpoint (e.g. LM Studio on host)
API_KEY=lm-studio                               # LLM auth
MODEL_NAME=qwen3-coder-next                     # resolved as ${MODEL_NAME} in workflow YAMLs
```

Change `BASE_URL` and `API_KEY` to switch providers (LM Studio, OpenAI, Gemini, etc.).
The backend proxies all LLM calls — the frontend never contacts the LLM directly.

## Taskfile

`DC` variable: `docker compose -f .setup/compose.yml --env-file .env`

`task up` runs `clean-workflows` after starting services — this deletes all sample
workflows shipped with ChatDev via `DELETE /api/workflows/{name}`. The team's
architect agent creates fresh, task-specific workflows from scratch.

## Volumes

All bind-mounted to `.data/` (visible on host filesystem):

| Path | Contents |
|---|---|
| `.data/warehouse/` | ChatDev-generated projects |
| `.data/output/` | Skill downloads (per session) |
| `.data/logs/` | Backend logs |
| `.data/data/` | vuegraphs.db |

## Resource Limits

- Backend: 2 CPUs, 4 GB RAM, 256 PIDs
