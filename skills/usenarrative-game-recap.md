---
name: Create a single-game recap
description: Generate a recap video for one game (project) from its OTIO-ready
  highlights, steering selection by duration, event types, or explicit highlight IDs.
api: openapi/usenarrative-sports-openapi-original.json
operations:
- GET /v1/tasks/{task_id}/info
- GET /v1/projects/{project_id}/highlight-options
- POST /v1/recaps
- GET /v1/recaps/{recap_id}
- GET /v1/recaps
- DELETE /v1/recaps/{recap_id}
generated: '2026-07-21'
method: generated
---

# Create a single-game recap

Authenticate with `Authorization: Bearer <API_KEY>`. Recaps require the
project's highlights to be OTIO-ready (current highlights generate `otio_edit`
automatically).

1. **Get the project ID** — `GET /v1/tasks/{task_id}/info` and read
   `task.project_id`; if `null`, retry after the task starts producing project
   data.
2. **Discover options** — `GET /v1/projects/{project_id}/highlight-options` to
   see the project's highlights and event types before steering selection.
3. **Create the recap** — `POST /v1/recaps` with the project ID and a target
   duration; optionally exclude event types or pass exact highlight IDs.
   Narrative selects highlights by event importance and editorial fit.
4. **Poll to completion** — `GET /v1/recaps/{recap_id}` until status moves
   through `queued` / `processing` to `complete` (or `failed`).
5. **Retrieve output** — the completed recap carries the merged EDL and the
   rendered video URL. `GET /v1/recaps` lists all recaps;
   `DELETE /v1/recaps/{recap_id}` removes one.

Rules: errors arrive as `{"detail": "<message>"}`; 429 signals the
concurrent-task limit; batch reads (`GET /v1/recaps/batch`) support ETag
conditional requests — send `If-None-Match` when polling collections.
