---
name: Build a multi-game highlight package
description: Assemble a highlight package spanning multiple games (projects) and
  retrieve its details as it renders.
api: openapi/usenarrative-sports-openapi-original.json
operations:
- GET /v1/projects/{project_id}/highlight-options
- POST /v1/highlight-packages
- GET /v1/highlight-packages/{package_id}
- GET /v1/highlight-packages
- GET /v1/highlight-packages/batch
- DELETE /v1/highlight-packages/{package_id}
generated: '2026-07-21'
method: generated
---

# Build a multi-game highlight package

Authenticate with `Authorization: Bearer <API_KEY>`.

1. **Discover per-project options** — for each source game, call
   `GET /v1/projects/{project_id}/highlight-options` to see available
   highlights, event types, and package creation options (for MMA this includes
   fight/count options).
2. **Create the package** — `POST /v1/highlight-packages` with the selected
   projects and options to assemble a multi-game package.
3. **Track progress** — `GET /v1/highlight-packages/{package_id}` returns
   package details and status; `GET /v1/highlight-packages` lists all packages.
4. **Poll efficiently** — `GET /v1/highlight-packages/batch` returns paginated
   package detail with ETag support; send `If-None-Match` to avoid re-fetching
   unchanged pages.
5. **Clean up** — `DELETE /v1/highlight-packages/{package_id}` removes a
   package.

Rules: errors arrive as `{"detail": "<message>"}`; there is no idempotency-key
contract, so reconcile with the list endpoint before retrying a create; retry
500s after a short delay.
