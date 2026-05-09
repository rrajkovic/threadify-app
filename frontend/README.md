# Discussion Thread Frontend

Next.js frontend for exploring discussion datasets served by the FastAPI
backend. The app provides a polished thread-map interface plus a prototype
analytics dashboard.

## Requirements

- Node.js and npm
- Backend running at `http://localhost:8000` by default

## Setup

```bash
npm install
npm run dev
```

Open `http://localhost:3000/user` for the main interface.

The prototype dashboard is available at `http://localhost:3000`.

## Environment

The frontend proxies backend calls through route handlers in `app/api`. Defaults
point to `http://localhost:8000`.

For a different backend URL, create or edit `.env.local`:

```bash
BACKEND_API_BASE_URL=http://localhost:8000
BACKEND_URL=http://localhost:8000
```

`BACKEND_API_BASE_URL` is used by dataset and raw message proxy routes.
`BACKEND_URL` is used by the annotated message and AI summary proxy routes.

## Available Routes

- `/user` - main Thread Map interface with map/chat modes, topic filters, time
  controls, upload, fullscreen map, detail sheets, and AI summaries
- `/` - prototype dashboard with chat view, message graph, topic graph, and
  sentiment/topic summaries
- `/test` - experimental variant of the user interface

## Backend Proxy Routes

| Frontend route | Backend route |
| --- | --- |
| `GET /api/datasets` | `GET /datasets` |
| `POST /api/datasets/upload` | `POST /datasets/upload` |
| `GET /api/discussions/{datasetId}/messages` | `GET /discussions/{dataset_id}/messages` |
| `GET /api/discussions/{datasetId}/messages/annotated` | `GET /discussions/{dataset_id}/messages/annotated` |
| `GET /api/discussions/{datasetId}/messages/ai-summary` | `GET /discussions/{dataset_id}/ai-summary` |

## Scripts

```bash
npm run dev    # Start local development server
npm run build  # Build for production
npm run start  # Start production server after build
npm run lint   # Run ESLint
```

## Main Packages

- Next.js 16 and React 19
- TypeScript
- Tailwind CSS 4
- `@xyflow/react` for the map canvas
- `dagre` and `elkjs` for graph layout support
- `lucide-react` for icons
- `motion` for interaction animation
