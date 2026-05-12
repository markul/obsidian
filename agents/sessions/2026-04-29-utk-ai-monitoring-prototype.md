---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-29
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-04-29 UTK AI Monitoring Prototype

## Goal

- Turn the `utk-ai` first slice into a usable local prototype for Kibana error monitoring with real MCP, AlfaGen, PostgreSQL persistence, and browser/API workflows

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[agents/sessions/2026-04-28-utk-ai-initial-plan|2026-04-28 UTK AI Initial Plan]]
  - [[daily/2026-04-29]]

## Actions

- Split local development so `skaffold dev -p deps --port-forward` runs only PostgreSQL and API/Web run locally through `dotnet watch` or `dotnet run`
- Removed namespace usage from the local/deps manifests to match the deployment style used by `product-change-workflow-service`
- Ran the local app against forwarded PostgreSQL on `127.0.0.1:5432` and kept it available at `http://127.0.0.1:5088`
- Added `ufr-kpnsb-product-request-workflow-service` as a monitored service
- Added a UI trigger button and environment/service dropdowns for manual monitoring runs
- Reworked the findings page from cards to a table and moved seen/cause/action/stacktrace content into a details modal
- Fixed Kibana representative links to use the working SKP Discover URL shape with the configured data view id and `_id` query
- Changed LLM output instructions so finding, cause, and action text is in Russian while technical identifiers remain unchanged
- Removed the large banner from the page
- Added stacktrace persistence in `error_evidence.stack_trace` and surfaced it in the details modal and evidence API
- Updated `utk-mcp` Kibana normalization to expose stack traces from structured stack fields, multiline `message`, plain `exception`, and plain `error`
- Added an exception-focused supplement query so monitoring samples include exception logs that can be older than the newest 100 error hits
- Verified `ufr-kpnsb-product-request-workflow-service` stacktraces for both embedded-message and `error`-field cases, including `FindThirdPartyLimitObjectsStep.GetCcObjectTypeInfoIdAsync(...):line 271`
- Changed rerun semantics: rerunning monitoring for the same UTC date, environment, and selected service replaces old findings/evidence for that scope instead of accumulating duplicate historical result rows
- Added `/api/error-findings` stacktrace output and wired filter changes to query stored results and redraw the table client-side
- Restarted the local `dotnet watch` app after Razor markup lag; verified the served page contains `data-filter-form`, `data-findings-table`, `data-findings-body`, and `data-details-modal`
- Converted the UI direction from Razor/server-rendered pages to a standalone Blazor WebAssembly SPA, modeled after `Markul.Web`'s standalone `Markul.Blazor` structure
- Renamed the ASP.NET Core host project from `Utk.Ai.Web` to `Utk.Ai.Api` and the Blazor WebAssembly project from `Utk.Ai.Blazor` to `Utk.Ai.Web`
- Decoupled `Utk.Ai.Api` from `Utk.Ai.Web`: removed the API project reference, hosted-WASM package, Blazor middleware/fallback, Razor pages, and API static UI assets
- Added standalone UI configuration via `src/Utk.Ai.Web/wwwroot/appsettings.json` with local `ApiBaseUrl=http://127.0.0.1:5088/`
- Stopped all local UTK AI background processes after validation: API, Web dev server, and `skaffold dev -p deps --port-forward`

## Decisions

- Use `skaffold dev -p deps --port-forward` for local dependencies only and run API/Web locally with `dotnet watch` or `dotnet run`
- Preserve historical monitoring only across different dates/environments/services; reruns for the same selected UTC date/environment/service should replace old scoped results
- Store stack traces in `error_evidence.stack_trace` and always show them in details when MCP/Kibana exposes them
- Query stored findings through the JSON API for filter changes instead of triggering a monitoring run
- Use standalone Blazor WebAssembly for the operator UI instead of continuing with Razor pages

## Validation

- `dotnet build Utk.Ai.slnx`
- `dotnet build` in `/home/marat/dev/git/alfa/utk-mcp`
- `docker compose up -d --build` in `/home/marat/dev/git/alfa/utk-mcp`
- `curl -fsS http://127.0.0.1:5088/health`
- `POST /api/monitoring-runs` for `2026-04-29/dev/ufr-kpnsb-product-request-workflow-service`
- `GET /api/error-findings?targetDate=2026-04-29&environment=dev&serviceName=ufr-kpnsb-product-request-workflow-service`
- `dotnet restore Utk.Ai.slnx --source https://api.nuget.org/v3/index.json --verbosity minimal`
- `dotnet build Utk.Ai.slnx --no-restore`
- `GET http://127.0.0.1:5088/` returns `404`, confirming the API no longer serves UI
- `GET /api/error-findings` with `Origin: http://localhost:5090` returns CORS headers for the standalone UI

## Current State

- Local deps Skaffold profile plus separate local API/Web `dotnet` runs are the current development path
- `utk-mcp` is running locally on `127.0.0.1:1985` and exposes normalized `stackTrace` for relevant Kibana hits
- No UTK AI local processes are currently running after the explicit stop
- The UI can trigger monitoring, query stored results, open representative Kibana Discover links, and show stacktrace details
- The current UI is implemented as `src/Utk.Ai.Web` and is configured for `http://localhost:5090`; `src/Utk.Ai.Api` is configured for `http://127.0.0.1:5088`
- Reruns for the same selected date/environment/service now replace prior scoped data
