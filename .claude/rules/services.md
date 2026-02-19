# Docker Services & Configuration

## Networking

| From → To | URL |
|---|---|
| Browser → Frontend | `http://localhost:5173` |
| Vite proxy (`/api`, `/ws`) → Backend | `http://backend:6400` (Docker internal) |
| Backend → Ollama | `http://ollama:11434/v1` (Docker internal) |

The frontend uses relative URLs (`/api/...`). Vite proxies them to the backend
container. The browser never talks to port 6400 directly.

## .env

```env
BASE_URL=http://ollama:11434/v1   # LLM endpoint (backend → Ollama, Docker internal)
API_KEY=ollama                     # LLM auth
MODEL_NAME=glm-4.7-flash          # resolved as ${MODEL_NAME} in workflow YAMLs
OLLAMA_HOST_PORT=11435             # host port (avoids conflict with host Ollama)
```

To use an external provider, change `BASE_URL` and `API_KEY` (e.g., OpenAI, Gemini).
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
| `.data/ollama/` | Model weights |
| `.data/warehouse/` | ChatDev-generated projects |
| `.data/output/` | Skill downloads (per session) |
| `.data/logs/` | Backend logs |
| `.data/data/` | vuegraphs.db |

## Resource Limits

- Ollama: 64 GB memory (large models)
- Backend: 2 CPUs, 4 GB RAM, 256 PIDs
