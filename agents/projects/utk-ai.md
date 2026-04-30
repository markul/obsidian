---
tags:
  - agents/project
note-type: agent-project
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
status: active
created: 2026-04-28T18:46
completed: 
related-ticket: 
---

# UTK AI

## Status

- `active`

## Timestamps

- Created: `2026-04-28 18:46`
- Completed: `not completed`

## Goal

- Plan and then implement `utk-ai` as a `.NET 10` web application for AI-assisted operational workflows, starting with Kibana error monitoring backed by `utk-mcp`, AlfaGen OpenAI-compatible models, and PostgreSQL

## Plan

1. Add `utk-mcp` Kibana log retrieval for configured service/environment targets.
2. Add AlfaGen OpenAI-compatible analysis flow and persist clustered findings/evidence.
3. Add scheduling and harden the development API/UI around real monitor output.

## Scope

- Repository: `utk-ai`
- Local path: `/home/marat/dev/git/alfa/utk-ai`
- Planned runtime: `.NET 10` ASP.NET Core API plus standalone Blazor WebAssembly UI
- Planned first feature: Kibana error monitoring with LLM aggregation, PostgreSQL persistence, and a standalone Blazor WebAssembly UI
- Planned local dev shape: Skaffold dependencies profile for PostgreSQL, with API and Web run locally
- Related notes:
  - [[agents/sessions/2026-04-28-utk-ai-bootstrap|2026-04-28 UTK AI Bootstrap]]
  - [[agents/sessions/2026-04-28-utk-ai-initial-plan|2026-04-28 UTK AI Initial Plan]]
  - [[agents/sessions/2026-04-29-utk-ai-monitoring-prototype|2026-04-29 UTK AI Monitoring Prototype]]

## Validation

- `dotnet build Utk.Ai.slnx`
- `skaffold render --offline=true`
- `skaffold dev --port-forward`
- `dotnet run --project src/Utk.Ai.Api/Utk.Ai.Api.csproj --no-build --urls http://127.0.0.1:5080`
- `curl -fsS http://127.0.0.1:5080/health`

## Current State

- Local Git repository initialized at `/home/marat/dev/git/alfa/utk-ai`
- Local machine has `.NET SDK 10.0.107` available
- Repo now contains `Utk.Ai.slnx` with `Domain`, `Application`, `Infrastructure`, `Api`, and `Web` projects targeting `net10.0`
- EF Core/PostgreSQL persistence is scaffolded with monitored targets, monitoring runs, findings, evidence, and an initial migration
- Skaffold local development now has a `deps` profile for PostgreSQL-only local dependencies
- API startup applies migrations only when `Skaffold__ApplyMigrations=true`, currently used for local development
- The web UI shows stored findings for a UTC date, environment, and service, defaulting to the previous UTC day
- Development JSON APIs exist for triggering monitor runs, listing runs, querying findings, and fetching evidence
- Current monitor trigger preserves run history, queries `utk-mcp` for sampled Kibana error logs, sends those logs to AlfaGen, and persists LLM-produced findings/evidence
- Repo docs define the product direction and architecture in `README.md` and `docs/initial-architecture.md`
- The first feature will monitor Kibana errors for configured service/environment targets, aggregate findings with an AlfaGen OpenAI-compatible model, persist results to PostgreSQL, and expose a date- and environment-filterable web page for stored findings
- The planned implementation now explicitly uses the official MCP C# SDK for `utk-mcp` communication and the official OpenAI .NET SDK for model communication
- Local development is now planned around repo-root Skaffold for dependencies plus local `dotnet` runs for API and Web
- Database migrations are planned to run automatically from the API during startup only in explicit local Skaffold mode rather than from a separate migration container
- Previous-day monitoring uses UTC day boundaries
- Manual reruns replace old findings/evidence for the same selected UTC date, environment, and service scope instead of accumulating duplicate stored results
- First live Kibana sample for `skp-product-change-workflow-service` in `dev` showed a dominant Kafka SASL auth failure pattern, so the first DB schema should retain source evidence plus parsed dependency fields such as dependency type, endpoint, component, error kind, failure point, normalized signature, and a representative Kibana URL for browser drill-down
- The first slice should expose JSON API endpoints to trigger monitoring runs, inspect run history/status, query findings, and fetch evidence so AI agents can test the app without browser automation
- Validation passed for `dotnet build Utk.Ai.slnx`, `skaffold render --offline=true`, app startup, `GET /health`, and index page loading without a running database
- `skaffold render --offline=true` validates the local manifests; local runtime currently uses `skaffold dev -p deps --port-forward` for PostgreSQL only
- AlfaGen app config now uses endpoint `https://alfagen.moscow.alfaintra.net/continue-dev`, model `MiniMaxAI/MiniMax`, and a secret-backed API key rather than a tracked key
- Local MCP config now points to remote enabled `utk-mcp` at `http://127.0.0.1:1985/mcp`
- Smoke test confirmed the API trigger reaches MCP and persists `totalHits=10000` plus `sampledHits=100`; full LLM path still requires providing `AlfaGen__ApiKey` to the running app
- Local smoke tests now have an ignored `.env.smoke` file on this machine and Kubernetes secret `skp/utk-ai-alfagen`; tracked scripts load/sync the key without committing it
- Trigger API now supports custom `dateFromUtc`/`dateToUtc` rolling windows in addition to UTC-day `targetDate`
- Rolling last-24-hours smoke test completed successfully for `2026-04-28T04:10:15Z` to `2026-04-29T04:10:15Z` with `sampledHits=100` and `findingsCount=1`
- Local development now uses `skaffold dev -p deps --port-forward` for PostgreSQL only and runs the app locally with `dotnet watch` on `http://127.0.0.1:5088`
- `ufr-kpnsb-product-request-workflow-service` is now monitored alongside `skp-product-change-workflow-service`
- `utk-mcp` Kibana normalization now exposes stack traces from structured stack fields, multiline `message`, plain `exception`, and plain `error`
- The app persists stack traces into `error_evidence.stack_trace` and shows them in the details modal and evidence API
- The findings UI uses a table with a details modal, a trigger button, environment/service dropdowns, representative Kibana links, and AJAX filtering through `/api/error-findings`
- The UI architecture has moved from Razor pages to a standalone Blazor WebAssembly client in `src/Utk.Ai.Web`; `src/Utk.Ai.Api` is API-only and does not reference or serve the UI
- All local UTK AI background processes were stopped after validation; API, Web, and deps Skaffold are not currently running

## Key Decisions

- Build the first release as an `ASP.NET Core net10.0` API app with a hosted background monitoring flow and a separate Blazor WebAssembly UI
- Use the existing `utk-mcp` server as the Kibana integration boundary
- Use the official MCP C# SDK for the `utk-mcp` client layer
- Use the official OpenAI .NET SDK for AlfaGen model access through the OpenAI-compatible API
- Use Skaffold for local development, borrowing the repo-root workflow shape from `skp-product-change-workflow-service`
- Keep the local dependency stack minimal, currently PostgreSQL only under the `deps` Skaffold profile
- Let the API own EF Core migration application during startup only for explicit local Skaffold mode
- Treat all reporting dates as UTC days
- Replace old stored results when rerunning the same selected UTC date, environment, and service scope
- Include development-friendly JSON API endpoints for triggering runs and querying persisted results
- Keep the LLM provider integration generic to an OpenAI-compatible AlfaGen endpoint and make the model configurable
- Use standalone Blazor WebAssembly for the operator UI, keeping `Utk.Ai.Api` API-only and `Utk.Ai.Web` as the SPA client with a configurable API base URL
- Use this project note as the durable home for future scope, validation, and implementation decisions

## Blockers

- No current blocker for manual trigger smoke path
