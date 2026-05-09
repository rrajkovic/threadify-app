# Discussion Thread Backend

FastAPI backend for loading, validating, parsing, annotating, and summarizing
threaded discussion datasets.

## Requirements

- Python 3.8+
- `pip`
- Optional: OpenAI API key for live annotation and summarization
- Optional: `jq` for the shell test scripts

## Setup

```bash
python3 -m venv venv
source venv/bin/activate
python -m pip install -r requirements.txt
bash run.sh
```

The API runs at `http://localhost:8000`.

Interactive docs are available at `http://localhost:8000/docs`.

You can also start the server directly:

```bash
python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

## Optional AI Configuration

Create `backend/.env`:

```bash
OPENAI_API_KEY=sk-your-key-here
```

When `OPENAI_API_KEY` is present, the backend calls OpenAI for topic labels,
sentiment labels, inferred reply links, summaries, and key points.

When the key is missing or a request fails, endpoints still return valid
responses with fallback values.

## Startup Behavior

On startup the backend:

- Initializes `backend/.cache/ai_cache.sqlite3`
- Lists file-backed datasets from `backend/data/*.json`
- Warms annotation and summary caches for built-in datasets in a background
  thread

Uploaded datasets are stored in process memory and are cleared when the backend
restarts.

## Endpoints

### Health

```http
GET /
```

Response:

```json
{
  "status": "ok",
  "service": "discussion-thread-backend"
}
```

### List Datasets

```http
GET /datasets
```

Returns sorted dataset IDs from `backend/data` plus any in-memory uploads.

```json
{
  "datasets": ["course_content", "discussion_demo", "project_qa"]
}
```

### Upload Dataset

```http
POST /datasets/upload
```

Request body:

```json
{
  "name": "custom_dataset",
  "messages": [
    {
      "id": "m1",
      "author": "Alice",
      "timestamp": "2026-03-01T09:00:00Z",
      "text": "Can we clarify the project deadline?",
      "parentId": null
    }
  ]
}
```

The backend validates each row, skips invalid rows and duplicate IDs, sorts
valid messages by timestamp, sanitizes the dataset name, and stores the result
in memory.

Response:

```json
{
  "datasetId": "custom_dataset",
  "messageCount": 1,
  "warnings": []
}
```

### Raw Messages

```http
GET /discussions/{dataset_id}/messages
```

Returns the validated flat message list.

```json
{
  "datasetId": "discussion_demo",
  "messageCount": 8,
  "messages": [
    {
      "id": "m1",
      "author": "Alice",
      "timestamp": "2024-01-15T10:30:00",
      "text": "Should we extend the deadline?",
      "parentId": null,
      "topic": "unknown",
      "sentiment": "unknown"
    }
  ],
  "warnings": []
}
```

### Thread Tree

```http
GET /discussions/{dataset_id}/thread
```

Builds parent/child thread trees from explicit `parentId` links.

```json
{
  "datasetId": "example",
  "roots": [
    {
      "id": "m1",
      "author": "Alice",
      "timestamp": "2026-03-01T09:00:00",
      "text": "Can we clarify the project deadline?",
      "parentId": null,
      "topic": "unknown",
      "sentiment": "unknown",
      "children": []
    }
  ],
  "orphans": [],
  "stats": {
    "messageCount": 1,
    "rootCount": 1,
    "orphanCount": 0,
    "maxDepth": 1
  },
  "warnings": []
}
```

### Annotated Messages

```http
GET /discussions/{dataset_id}/messages/annotated
```

Returns messages enriched with AI/fallback fields:

- `topic`
- `sentiment`
- `inferredReplyToId`
- `replyInferred`

```json
{
  "datasetId": "discussion_demo",
  "messageCount": 8,
  "messages": [
    {
      "id": "m1",
      "author": "Alice",
      "timestamp": "2024-01-15T10:30:00",
      "text": "Should we extend the deadline?",
      "parentId": null,
      "topic": "deadline",
      "sentiment": "neutral",
      "inferredReplyToId": null,
      "replyInferred": false
    }
  ],
  "warnings": []
}
```

### AI Summaries

```http
GET /discussions/{dataset_id}/ai-summary
```

Returns one summary per root thread:

```json
{
  "datasetId": "discussion_demo",
  "summaryCount": 2,
  "summaries": [
    {
      "root_id": "m1",
      "main_topic": "deadline",
      "summary": "The thread discusses whether a deadline should be extended.",
      "key_points": [
        "Some participants support an extension.",
        "Others raise fairness concerns.",
        "A short compromise extension is suggested."
      ]
    }
  ],
  "warnings": []
}
```

## Data Format

Datasets are JSON arrays stored in `backend/data`.

```json
[
  {
    "id": "m1",
    "author": "Alice",
    "timestamp": "2026-03-01T09:00:00Z",
    "text": "Message content",
    "parentId": null,
    "topic": "deadline",
    "sentiment": "neutral"
  }
]
```

Required fields:

- `id`
- `author`
- `timestamp`
- `text`

Optional fields:

- `parentId`
- `topic` defaults to `unknown`
- `sentiment` defaults to `unknown`

## Built-In Datasets

- `course_content` - 9 messages
- `course_difficulty` - 9 messages
- `discussion_demo` - 8 messages
- `large_discussion_test_dataset` - 431 messages
- `project_qa` - 9 messages
- `resources` - 9 messages
- `test_upload` - 9 messages

## Caching

The AI service stores cache entries in SQLite at:

```text
backend/.cache/ai_cache.sqlite3
```

The cache includes per-message annotations, per-root summaries, full annotated
datasets, and full summary responses. Cache files are local runtime artifacts and
are ignored by git.

## Testing

Start the backend first, then run:

```bash
./test_ai_endpoints.sh
./test_ai_comprehensive.sh
```

Both scripts call the local API at `http://localhost:8000` and pretty-print
responses with `jq`.

## Error Handling

- Missing datasets return `404`
- Invalid dataset JSON returns `400`
- Invalid rows inside a dataset are skipped and reported in `warnings`
- Duplicate message IDs are skipped and reported in `warnings`
- AI failures fall back to `unknown` labels or fallback summaries instead of
  failing the endpoint
