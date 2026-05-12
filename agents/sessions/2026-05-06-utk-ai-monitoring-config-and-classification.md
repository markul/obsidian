---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-06
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-06 UTK AI Monitoring Config And Classification

## Goal

- Keep the local `utk-ai` monitoring app usable for broader SKP/KPNSB service analysis and correct classification of domain validation failures.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[daily/2026-05-06]]
  - [[agents/sessions/2026-05-05-utk-ai-aggregation-persistence|2026-05-05 UTK AI Aggregation Persistence]]
  - [[agents/sessions/2026-05-05-utk-mcp-kibana-aggregation|2026-05-05 UTK MCP Kibana Aggregation]]

## Actions

- Added all 30 `dev` services from the KPNSB/SKP service list to `src/Utk.Ai.Api/appsettings.json` under `Monitoring:Targets`.
- Updated the Blazor service dropdown in `src/Utk.Ai.Web/Pages/Findings.razor` to expose the same 30 configured services.
- Restarted the API watcher so `ConfiguredTargetSeeder` inserted the new monitored targets into local PostgreSQL.
- Corrected finding category normalization in `AlfaGenLogFindingAnalyzer`: evidence containing `ValidationException`, `validation_error`, or `Не найдены субъекты` now resolves to category `application`, even when the same log also has outbound HTTP `400` context.
- Preserved outbound HTTP dependency context for application validation errors, so details can still show the dependency endpoint while the root category remains `application`.
- Stopped the local UTK AI API, Web, and Skaffold/PostgreSQL stack after validation while leaving the Docker `utk-mcp` container running on `127.0.0.1:1985`.

## Decisions

- Keep category classification primarily model-driven through AlfaGen, but apply deterministic overrides for known domain validation markers.
- Keep the API config and Web dropdown service lists in sync until the UI grows an API endpoint for fetching monitored targets dynamically.
- Treat HTTP status as transport context when stack trace or message clearly identifies an application/domain validation exception.

## Validation

- `dotnet build src/Utk.Ai.Api/Utk.Ai.Api.csproj`
- `jq '.Monitoring.Targets | length' src/Utk.Ai.Api/appsettings.json` returned `30`.
- Local PostgreSQL query confirmed `30` enabled `dev` monitored targets after API startup.
- `curl` health checks returned `200` for API, Web, and MCP before the explicit stack stop.
- Blazor current `Utk.Ai.Web*.wasm` asset returned `200` before the explicit stack stop.

## Current State

- Docker `utk-mcp` remains running on `127.0.0.1:1985`.
- UTK AI API, Web, and Skaffold/PostgreSQL dependencies were intentionally stopped after validation.
- Monitoring target configuration contains 30 enabled `dev` services, including `skp-product-change-workflow-service`, `skp-utk-blocking-service`, and the KPNSB service set from the UI service list.
