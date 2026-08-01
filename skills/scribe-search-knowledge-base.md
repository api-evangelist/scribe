---
name: Search the Scribe knowledge base
description: Full-text search across an organization's Scribe Documents and Knowledge Pages, scoped by team, then open the best match.
api: openapi/scribe-search-retrieval-openapi.yml
operations: [v1_teams_list, v1_search_documents, v1_retrieve_document]
---

# Search the Scribe knowledge base

Use the Scribe Search & Retrieval API to find process documentation and reference
pages, then read the full content of the best match. The API is read-only.

## Prerequisites
- An Enterprise-plan API key. Generate it at Settings > Developer Access
  (`https://scribehow.com/settings?tab=developerAccess`); the full key is shown once.
- Send it on **every** request as `X-API-Key: sc_<id>.<secret>`.
- Base URL: `https://public-api.scribehow.com`.

## Steps
1. **Discover teams** — call `v1_teams_list` (`GET /v1/teams/`). Each key is scoped to
   one or more teams; capture the `id` values to use as `team_ids` filters.
2. **Search** — call `v1_search_documents` (`POST /v1/search/`) with a JSON body:
   `{ "search": "<query>", "type": "scribe" | "page" (optional), "team_ids": [...] (optional),
   "sort_by": "relevance", "limit": 15 }`. `search` is required. This endpoint is **not
   paginated** and returns at most 20 results (default 15).
3. **Open a result** — take the `id` of the best hit and call `v1_retrieve_document`
   (`GET /v1/documents/{id}/`). For `type: scribe`, read the `actions[]` steps (each may
   have a `screenshot_url`); for `type: page`, read the Editor.js `content` object.

## Rules
- Auth: static API key in `X-API-Key` — no OAuth, no tokens to refresh.
- Rate limits: 300 req/min combined for search+teams; back off on `429`
  (the `detail` message states seconds until retry). See `rate-limits/scribe-rate-limits.yml`.
- Errors: `401` invalid/missing key, `403` plan lacks Public API access, `429` throttled.
  Envelope is `{ "detail": "<message>" }`. See `errors/scribe-problem-types.yml`.
- Do not attempt writes — the Public API exposes no create/update/delete operations.
