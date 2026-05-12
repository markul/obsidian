---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-06
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-06 UTK AI Investigation Agent And Plain Search

## Goal

- Add an AlfaGen-backed investigation workflow for monitoring issues and a transient Kibana-link investigation path that does not persist results.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[daily/2026-05-06]]
  - [[agents/sessions/2026-05-06-utk-ai-monitoring-config-and-classification|2026-05-06 UTK AI Monitoring Config And Classification]]

## Actions

- Added a custom monitoring issue investigator backed by AlfaGen through the existing OpenAI-compatible model integration.
- Made persisted monitoring issue investigations queueable: API requests create investigation jobs, a hosted worker processes `New` and resumable `Pending` jobs, and results are stored after completion.
- Added SignalR progress reporting for queued investigations so the Web UI can update while the background worker searches logs, calls AlfaGen, and persists the result.
- Added request/response logging around AlfaGen issue-investigation calls with sensitive configuration kept out of tracked files.
- Validated that `loanmanager-signatory-service` logs are available through `utk-mcp`; a live replay for `2026-05-06` found duplicate-key `IX_rlClientSignatoryABS_bMain_1_Unique` errors that the investigator should surface.
- Expanded investigator follow-up search to allow dependency-service log searches, fixing the earlier false claim that `loanmanager-signatory-service` logs were unavailable.
- Added a non-persistent `POST /api/plain-investigations` endpoint that accepts a Kibana link, derives environment/service/time window where possible, searches logs through `utk-mcp`, and returns an AlfaGen investigation result without writing DB records.
- Added a `Plain search` Web page at `/plain-search` for Kibana-link investigations.
- Reworked the Blazor layout to use a left sidebar with `Monitoring` and `Plain search` navigation, with the theme switcher pinned at the bottom.
- Cleaned local UTK AI terminal processes and restarted only API and Web using stable `dotnet run`, not `dotnet watch`.
- Fixed plain Kibana-link parsing for Discover URLs with relative time and field filters: `now-24h/h` windows are resolved, `requestId`/`traceId` filters become focused Lucene queries, and exact focused searches no longer fall back to unrelated all-error service logs.
- Converted plain Kibana-link investigations from transient synchronous calls into persisted queued jobs with a generated ID, stored original link, parsed search context, raw MCP search JSON, final AlfaGen result JSON, status/timestamps, error message, worker processing, and SignalR/polling progress in the Web UI.
- Added Plain search investigation history: `GET /api/plain-investigations?limit=...` returns recent persisted jobs, and the Web page shows a history panel where selecting an item loads its stored full result by ID.
- Added configurable corporate VPN preflight before every `utk-ai` Kibana MCP call: `Mcp:VpnPreflight` defaults to `snxctl status`, requires `Connected since:`, and can be disabled or adjusted in config.

## Decisions

- Keep persisted monitoring issue investigations durable and queue-backed because they belong to stored monitoring findings and should survive UI refreshes.
- Plain Kibana-link investigations started transient, then moved to persisted queued jobs after the user requested durable link/result storage for later debugging.
- Continue using `utk-mcp` as the Kibana boundary for both persisted monitoring and plain investigations.
- Do not use hot reload or `dotnet watch` for the current local runtime; run API and Web with stable `dotnet run`.

## Validation

- `dotnet build Utk.Ai.slnx`
- `curl -sS --max-time 3 http://127.0.0.1:9596/health` returned `Healthy`.
- `curl -sS -I --max-time 3 http://127.0.0.1:9595/plain-search` returned `HTTP/1.1 200 OK`.
- Invalid `POST /api/plain-investigations` with an empty Kibana URL returned `400` and `{"error":"Kibana URL is required."}`.
- Process check showed the current repo running only API and Web `dotnet run` commands, with expected child processes for the API executable and Blazor dev server.
- Parser reflection check for the reported Discover URL derived service `ufr-kpnsb-product-request-workflow-service`, query `requestId:"0HNL6JGNCQ99C:00000001"`, and a `now-24h/h` to `now` UTC window.
- `dotnet build Utk.Ai.slnx` passed after adding `plain_kibana_investigations` persistence and the queued plain investigation worker.
- `POST /api/plain-investigations` with an empty URL returned `400`; a valid Discover URL returned `202 Accepted` with persisted ID `0ad804c8-612f-4af6-b9ef-a981d15c5426`, and `GET /api/plain-investigations/{id}` returned parsed service `ufr-kpnsb-product-request-workflow-service`, query `requestId:"0HNL6JGNCQ99C:00000001"`, the resolved UTC window, zero hits, and an error status.
- Investigated plain search ID `a5d92d8d-4eea-4a82-b0e5-17d47cd6c692`: initial persisted MCP payload had `total=0` for exact `requestId:"0HNL6JGNCQ99C:00000001"` because `utk-mcp` did not include top-level `requestId`/`traceId` fields in its Kibana query/normalization allowlist. Patched `utk-mcp`, rebuilt the Docker container, changed `utk-ai` plain-link exact-field queries to search quoted field values, and requeued the same ID. The retry found two hits for request `0HNL6JGNCQ99C:00000001`, including the `skpdb-dev:3225/api/v1/uid/deal-uid` HTTP 500 and upstream stack trace through `LoanManagerUidService.GetDealUidAsync` and `FindDealUidsStep`.
- `dotnet build Utk.Ai.slnx` passed after adding history. API/Web were restarted; `GET /api/plain-investigations?limit=5` returned recent records, and ID `a5d92d8d-4eea-4a82-b0e5-17d47cd6c692` completed with stored result, `totalHits=2`, and high-confidence AlfaGen summary pointing to `loanmanager-uid-service` / `LMDealInfoODataProvider.IsUidRequiredForDealAsync`.
- `dotnet build Utk.Ai.slnx` passed after VPN preflight. With current `snxctl status` returning `Disconnected`, a direct MCP client check failed before MCP with `Corporate VPN preflight failed... did not report 'Connected since:'`; API restarted and `/health` stayed healthy.

## Current State

- API is running from `src/Utk.Ai.Api` with plain `dotnet run` on `http://localhost:9596`.
- Web is running from `src/Utk.Ai.Web` with plain `dotnet run` on `http://localhost:9595`.
- The Web app has primary routes for `Monitoring` at `/`, `Plain search` at `/plain-search`, and `Chat` at `/chat`.
- Plain Kibana-link investigation is now persisted and debuggable by ID; after the `utk-mcp` request/trace field fix and quoted-value query change, ID `a5d92d8d-4eea-4a82-b0e5-17d47cd6c692` completed with two hits and a stored AlfaGen result.
