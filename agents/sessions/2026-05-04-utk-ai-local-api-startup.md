---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-04
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-04 UTK AI Local API Startup

## Goal

- Make `Utk.Ai.Api` run locally in the intended developer workflow, especially `cd src/Utk.Ai.Api && dotnet watch run`

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Service note: [[work/alfa-bank/services/utk-ai|utk-ai]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[daily/2026-05-04]]

## Actions

- Added Development-only repo-root `.env` fallback loading in the API so ignored local AlfaGen settings can bind without manual export; keys such as `AlfaGen__ApiKey` are normalized to ASP.NET configuration keys.
- Moved migration startup configuration out of `Skaffold` and into `Database:ApplyMigrations`, defaulting to `true`.
- Kept `ApplyMigrations` configurable through appsettings/environment variables; `Database__ApplyMigrations=false` skips EF migration checks while the API still seeds configured monitoring targets against an existing schema.
- Removed the old `Skaffold__ApplyMigrations` deployment setting from local Skaffold manifests and updated repo docs.
- Disabled API launch-profile browser launching so `dotnet watch run` no longer tries to execute `http://localhost:9596` as a process.
- Diagnosed the remaining `dotnet watch run` failure as this machine's low `fs.inotify.max_user_instances=128`; the session cannot raise it without sudo.
- Added `export DOTNET_USE_POLLING_FILE_WATCHER=true` to `/home/marat/.zshrc`, so fresh interactive zsh terminals can run plain `dotnet watch run`.
- Investigated the UI error `utk-mcp search_kibana_logs failed: An error occurred invoking 'search_kibana_logs'.`; `utk-mcp` container logs showed DNS failure for short host `kibana-test`.
- Updated ignored local `/home/marat/dev/git/alfa/utk-mcp/.env` to use `UTK_MCP_Kibana__BaseUrl=https://kibana-test.moscow.alfaintra.net/s/skp/` and recreated the `utk-mcp` Docker container.

## Decisions

- Keep API migration startup app-owned and enabled by default, but make it configurable under `Database` rather than `Skaffold`.
- Prefer the literal local workflow `cd src/Utk.Ai.Api && dotnet watch run`; the machine-level zsh export handles polling watcher mode.
- Do not restrict API project-reference watching to work around inotify limits, because that would make hot reload less useful for application/infrastructure/domain edits.
- Use the FQDN Kibana host in containerized `utk-mcp` local config because Docker's resolver does not apply the host VPN search domain to the short `kibana-test` name.

## Validation

- `dotnet build Utk.Ai.slnx`
- `dotnet run --project src/Utk.Ai.Api/Utk.Ai.Api.csproj --no-build --urls http://127.0.0.1:9596`
- `Database__ApplyMigrations=false dotnet run --project src/Utk.Ai.Api/Utk.Ai.Api.csproj --no-build --urls http://127.0.0.1:9596`
- Fresh interactive zsh: `cd /home/marat/dev/git/alfa/utk-ai/src/Utk.Ai.Api && dotnet watch run`
- `curl -i http://localhost:9596/health` returned `200 Healthy`
- `docker exec utk-mcp-utk-mcp-1 getent hosts kibana-test.moscow.alfaintra.net`
- `POST /api/monitoring-runs` for `2026-05-03/dev/skp-product-change-workflow-service` completed successfully after the FQDN change
- `GET /api/error-findings?targetDate=2026-05-03&environment=dev&serviceName=skp-product-change-workflow-service` returned the persisted Kafka SASL auth finding

## Current State

- In new interactive zsh terminals, `dotnet watch run` from `src/Utk.Ai.Api` starts the API and prints `Polling file watcher is enabled`.
- API listens on `http://localhost:9596` via launch settings.
- Local Dockerized `utk-mcp` now resolves Kibana through `kibana-test.moscow.alfaintra.net`; the previous short host failed inside the container even while host VPN DNS was connected.
- No UTK AI API process was intentionally left running after validation.
