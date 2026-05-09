# Discussion Thread Analysis Platform

A full-stack project for loading, analyzing, and visualizing threaded discussion
datasets. The backend exposes a FastAPI API for dataset loading, thread parsing,
AI-assisted topic/sentiment annotation, summary generation, and in-memory uploads.
The frontend is a Next.js application with map and chat views for exploring those
threads.

## Project Structure

```text
.
|-- backend/
|   |-- app/
|   |   |-- main.py        # FastAPI app and routes
|   |   |-- loader.py      # JSON dataset loading and validation
|   |   |-- parser.py      # Parent/child thread tree construction
|   |   |-- schemas.py     # Pydantic response models
|   |   `-- ai_service.py  # OpenAI annotation, summaries, and SQLite cache
|   |-- data/              # Built-in JSON datasets
|   |-- requirements.txt
|   |-- run.sh
|   `-- README.md
|-- frontend/
|   |-- app/
|   |   |-- api/           # Next.js proxy routes to the backend
|   |   |-- components/    # Thread map, chat, filters, and detail sheets
|   |   |-- page.tsx       # Prototype analytics dashboard
|   |   `-- user/page.tsx  # Main polished thread-map interface
|   |-- package.json
|   `-- README.md
`-- run-dev.sh             # macOS helper that opens backend and frontend terminals
```

## Quick Start

### Backend

```bash
cd backend
python3 -m venv venv
source venv/bin/activate
python -m pip install -r requirements.txt
bash run.sh
```

The API runs at `http://localhost:8000`.

Interactive API docs are available at `http://localhost:8000/docs`.

Optional AI features use an OpenAI API key. Create `backend/.env` and add:

```bash
OPENAI_API_KEY=sk-your-key-here
```

Without an API key, the API still runs. Annotation fields may fall back to
`unknown`, and summaries use fallback text when live generation is unavailable.

### Frontend

```bash
cd frontend
npm install
npm run dev
```

Open `http://localhost:3000/user` for the main thread-map UI.

The development dashboard is available at `http://localhost:3000`.

If the backend is not on `http://localhost:8000`, set these in
`frontend/.env.local`:

```bash
BACKEND_API_BASE_URL=http://localhost:8000
BACKEND_URL=http://localhost:8000
```

### Start Both Apps on macOS

```bash
./run-dev.sh
```

This helper starts the backend from `backend/venv` and the frontend in separate
Terminal windows.

## Current Features

- Built-in datasets loaded from `backend/data/*.json`
- Custom JSON dataset upload through `POST /datasets/upload`
- Flat message list and hierarchical thread tree endpoints
- AI-assisted topic and sentiment annotation
- AI-assisted thread summaries and key points
- AI-inferred reply links for messages that look like contextual replies
- SQLite-backed AI cache in `backend/.cache/ai_cache.sqlite3`
- Background warmup for built-in datasets when the backend starts
- Next.js proxy routes so the browser can call the backend through the frontend
- Main UI with map/chat modes, dataset selection, JSON upload, topic filters,
  time grouping, fullscreen map view, message detail sheets, and summary footer

## API Overview

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `GET` | `/` | Health check |
| `GET` | `/datasets` | List built-in and uploaded datasets |
| `POST` | `/datasets/upload` | Upload an in-memory dataset |
| `GET` | `/discussions/{dataset_id}/messages` | Return validated messages |
| `GET` | `/discussions/{dataset_id}/thread` | Return root threads, orphans, and stats |
| `GET` | `/discussions/{dataset_id}/messages/annotated` | Return AI-enriched messages |
| `GET` | `/discussions/{dataset_id}/ai-summary` | Return summaries for root threads |

Frontend proxy routes mirror the backend under `frontend/app/api`, including
`/api/datasets`, `/api/datasets/upload`,
`/api/discussions/{datasetId}/messages`,
`/api/discussions/{datasetId}/messages/annotated`, and
`/api/discussions/{datasetId}/messages/ai-summary`.

## Data Format

Datasets are JSON arrays of message objects:

```json
[
  {
    "id": "m1",
    "author": "Alice",
    "timestamp": "2026-03-01T09:00:00Z",
    "text": "Should we extend the deadline?",
    "parentId": null,
    "topic": "deadline",
    "sentiment": "neutral"
  }
]
```

Required fields are `id`, `author`, `timestamp`, and `text`.

Optional fields are `parentId`, `topic`, and `sentiment`. The annotated endpoint
can also return `inferredReplyToId` and `replyInferred` for AI-inferred links.

## Built-In Datasets

The backend currently includes:

- `course_content` - 9 messages
- `course_difficulty` - 9 messages
- `discussion_demo` - 8 messages
- `large_discussion_test_dataset` - 431 messages
- `project_qa` - 9 messages
- `resources` - 9 messages
- `test_upload` - 9 messages

The repository root also contains additional standalone JSON discussion samples
that can be uploaded through the frontend.

## Testing

Backend endpoint scripts require the backend to be running and `jq` to be
installed:

```bash
cd backend
./test_ai_endpoints.sh
./test_ai_comprehensive.sh
```

Frontend checks:

```bash
cd frontend
npm run lint
npm run build
```

## Tech Stack

- Backend: FastAPI, Uvicorn, Pydantic, python-dateutil, OpenAI SDK,
  python-dotenv, SQLite
- Frontend: Next.js 16, React 19, TypeScript, Tailwind CSS 4, React Flow,
  dagre, elkjs, lucide-react, Motion

## Notes

- `.env`, `.env.local`, virtual environments, `node_modules`, `.next`, and AI
  cache files are ignored by git.
- Uploaded datasets are stored in backend memory and disappear when the backend
  restarts.
- `backend/Dockerfile` is currently empty; run the backend locally with `run.sh`.
