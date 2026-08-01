---
name: Sync Scribe documents into an external system
description: Incrementally list all accessible Scribe Documents and Knowledge Pages by update time and retrieve their full content for indexing or mirroring.
api: openapi/scribe-search-retrieval-openapi.yml
operations: [v1_teams_list, v1_get_documents, v1_retrieve_document]
---

# Sync Scribe documents into an external system

Mirror an organization's Scribe knowledge base into a search index, data warehouse,
or RAG store by paginating the document list and fetching each document's body.

## Prerequisites
- Enterprise-plan API key sent as `X-API-Key` on every request.
- Base URL: `https://public-api.scribehow.com`.

## Steps
1. **(Optional) Scope by team** — call `v1_teams_list` (`GET /v1/teams/`) to get `team_ids`.
2. **Page the catalog** — call `v1_get_documents` (`GET /v1/documents/`) with
   `limit=100&offset=0`, then increment `offset` by 100 until the response `next` is null.
   Order is newest-first by creation date. Optional filters: `type`, `team_ids`,
   `created__gte/lte`, `updated__gte/lte`.
3. **Incremental sync** — on later runs pass `updated__gte=<last-run ISO timestamp>` to fetch
   only documents with a meaningful user edit since your last sync. (Note: `updated__gte`
   targets the "last meaningful edit" column, distinct from the response `updated_at`.)
4. **Fetch bodies** — for each result `id`, call `v1_retrieve_document`
   (`GET /v1/documents/{id}/`). Store `actions[]` (Scribe Documents) or `content`
   (Knowledge Pages), plus `name`, `description`, `link`, `team`, and `user_owner`.

## Rules
- Pagination is limit/offset with a `{ count, next, previous, results }` envelope; max page size 100.
- Rate limits: document endpoints share 60 req/min — throttle your paging loop; back off on `429`.
- Read-only: there is no write-back path; treat Scribe as the source of truth.
- See `conventions/scribe-conventions.yml` for filtering/pagination detail and
  `errors/scribe-problem-types.yml` for the error envelope.
