---
name: Process a live sports stream into highlights
description: Create a Narrative Sports task, stream live video to its SRT URL, and
  retrieve rendered highlight clips with signed download URLs.
api: openapi/usenarrative-sports-openapi-original.json
operations:
- POST /v1/tasks/create-task
- GET /v1/tasks/{task_id}/info
- GET /v1/highlights
- GET /v1/highlights/{highlight_id}/signed-url
- POST /v1/highlights/batch-signed-urls
- POST /v1/tasks/{task_id}/kill
generated: '2026-07-21'
method: generated
---

# Process a live sports stream into highlights

Authenticate every request with `Authorization: Bearer <API_KEY>` (keys are
prefixed `sk-nr-`, managed at https://app.narrative-sports.com/api-dashboard).
The error envelope is `{"detail": "<message>"}`; 422 carries FastAPI-style
validation details.

1. **Create the task** — `POST /v1/tasks/create-task` with at least `{"sport":
   "soccer"}` (or `"mma"`). Pass `render_videos.render_video = "ffmpeg"` so
   clips are rendered. The call blocks until the worker publishes an SRT URL
   and returns `task_id` + `srt_url`.
2. **Stream to the SRT URL** — send the stream at 1x real-time speed using the
   `srt_url` exactly as returned (it embeds the per-task host and encryption
   passphrase). Source must be H.264/AVC over MPEG-TS at >= 3000 kb/s with
   clear spoken commentary audio.
3. **Watch task state** — `GET /v1/tasks/{task_id}/info`; `task_status` moves
   through `queued` / `running` / `ready` / `completed` (`error`, `failed`,
   `stopped` are terminal failures/stops). Use `task.project_id` for recap and
   package flows once it is non-null.
4. **List highlights** — `GET /v1/highlights?task_id=<task_id>` returns
   detected highlight metadata (event type; for MMA also `fight_number`,
   `round_number`, `fight_name`).
5. **Download clips** — `GET /v1/highlights/{highlight_id}/signed-url` for one
   video, or `POST /v1/highlights/batch-signed-urls` for many. URLs are
   time-limited.
6. **Stop the task** — `POST /v1/tasks/{task_id}/kill` when the game ends
   (graceful by default). Tasks also auto-stop at their `max_life_hours`;
   extend with `POST /v1/tasks/{task_id}/extend-life` if needed.

Rules: a 429 means your concurrent-task limit is reached — kill a task or wait;
retry 500s after a short delay; do not use the deprecated
`/v1/tasks/warmup` + `/v1/tasks/assign-task` flow for new integrations; no
idempotency keys exist, so avoid blind retries of `POST /v1/tasks/create-task`
(list tasks first to reconcile).
