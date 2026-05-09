# Available Discussion Datasets

This directory contains JSON datasets loaded automatically by the backend. Each
file name, without `.json`, becomes a dataset ID returned by `GET /datasets`.

## Current Datasets

| Dataset ID | Messages | Description |
| --- | ---: | --- |
| `course_content` | 9 | Course content and curriculum discussion |
| `course_difficulty` | 9 | Course difficulty, resources, and support discussion |
| `discussion_demo` | 8 | Deadline extension and grading policy demo |
| `large_discussion_test_dataset` | 431 | Larger stress-test discussion dataset |
| `project_qa` | 9 | Final project questions and answers |
| `resources` | 9 | Learning resources and tooling recommendations |
| `test_upload` | 9 | Small upload/test dataset |

## Format

Each dataset is a JSON array of message objects:

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

Required fields are `id`, `author`, `timestamp`, and `text`.

Optional fields are `parentId`, `topic`, and `sentiment`. Missing `topic` and
`sentiment` values default to `unknown`.

## Adding a Dataset

1. Add a new `.json` file to this directory.
2. Make sure the top-level value is an array of message objects.
3. Restart the backend or request `GET /datasets`; the loader will pick up the
   file name as the dataset ID.

Custom datasets can also be uploaded at runtime through the frontend or through
`POST /datasets/upload`. Runtime uploads are stored in memory and disappear when
the backend restarts.
