---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-28
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
related-project: "[[agents/projects/utk-ai|UTK AI]]"
related-ticket: 
---

# 2026-04-28 UTK AI Initial Plan

## Goal

- Define the first real product and architecture plan for `utk-ai`

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[agents/sessions/2026-04-28-utk-ai-bootstrap|2026-04-28 UTK AI Bootstrap]]
  - [[daily/2026-04-28]]

## Actions

- Re-read the bootstrap repo docs and the durable `UTK AI` project note
- Verified the local machine has `.NET SDK 10.0.107`
- Defined the first feature as Kibana error monitoring for configurable service/environment targets
- Chose the initial product shape: single ASP.NET Core `.NET 10` web app, hosted background monitoring flow, PostgreSQL persistence, and a server-rendered page for daily findings
- Recorded the initial architecture in `/home/marat/dev/git/alfa/utk-ai/docs/initial-architecture.md`
- Updated the repo `README.md` and `AGENTS.md` to reflect the chosen direction
- Updated the durable `UTK AI` project note with the new scope, plan, and open blockers
- Refined the implementation constraint to use the official MCP C# SDK for `utk-mcp` communication and the official OpenAI .NET SDK for AlfaGen model communication
- Checked the local-development example in `skp-product-change-workflow-service` to align on a repo-root Skaffold workflow
- Refined the local-dev plan to use Skaffold with only app and PostgreSQL containers and app-owned automatic migrations at startup
- Clarified that previous-day monitoring uses UTC day boundaries
- Clarified that manual reruns preserve run history by creating new monitoring runs
- Clarified that automatic EF Core migrations are only for explicit local Skaffold mode
- Sampled the last 24 hours of `dev` Kibana errors for `skp-product-change-workflow-service`; the dominant pattern was Kafka SASL authentication failure with a `3/3 brokers are down` downstream symptom
- Updated the architecture data model to include run query totals, source evidence identifiers, normalized messages, dependency metadata, failure point, error kind, recommended action, and a representative Kibana URL field
- Added the first-slice JSON API plan for triggering monitoring runs, inspecting run status/history, querying findings, and fetching evidence for AI-agent testability
- Implemented the first scaffold slice in `/home/marat/dev/git/alfa/utk-ai`
- Created `Utk.Ai.slnx` with `Utk.Ai.Domain`, `Utk.Ai.Application`, `Utk.Ai.Infrastructure`, and `Utk.Ai.Web`
- Added EF Core/PostgreSQL entities and initial migration for monitored targets, run history, error findings, and evidence
- Added package references for `ModelContextProtocol` and `OpenAI`; real MCP/Kibana and AlfaGen calls remain the next integration pass
- Added the Razor findings page with UTC date, environment, and service filters
- Added development JSON API endpoints for trigger/list/get monitoring runs, list/get findings, and list evidence
- Added Skaffold local stack with app plus PostgreSQL containers and app-owned migrations gated by `Skaffold__ApplyMigrations=true`
- Updated repo `README.md` and `AGENTS.md` from plan-only state to current implementation state
- Validated `dotnet build Utk.Ai.slnx`, `skaffold render --offline=true`, app startup, `GET /health`, and index page load without a running database
- Ran `skaffold dev --port-forward` end-to-end and fixed issues found by the run: incorrect Alfa Docker image path, incompatible SDK pin for the build container, missing `skp` namespace, and app migration startup racing Postgres readiness
- Verified the Skaffold stack reached `2/2 Running`, `GET /health` returned `Healthy`, the index page returned `200`, `POST /api/monitoring-runs` created a run for `dev/skp-product-change-workflow-service`, and `GET /api/monitoring-runs` returned that run
- Wired AlfaGen endpoint/model into tracked config and local Skaffold env, with API key expected from Kubernetes secret `utk-ai-alfagen` instead of tracked files
- Wired local MCP config to remote enabled `utk-mcp` at `http://127.0.0.1:1985/mcp`
- Implemented actual API trigger integration: official MCP C# SDK client calls `search_kibana_logs`, OpenAI .NET SDK sends sampled logs to AlfaGen, and returned clusters are persisted as findings/evidence
- Smoke tested trigger without a model key: app applied migrations against temporary Postgres, reached MCP, persisted a failed run with `totalHits=10000` and `sampledHits=100`, then failed at the expected `AlfaGen API key is not configured` check
- Added `.env.smoke.example`, `scripts/sync-local-secrets.sh`, and `scripts/smoke-test-monitoring.sh`; created ignored local `.env.smoke` and synced Kubernetes secret `skp/utk-ai-alfagen` on this machine
- Added optional trigger `dateFromUtc`/`dateToUtc` support and updated smoke script to run a real rolling last-24-hours window
- Ran `UTK_AI_SMOKE_LAST_24_HOURS=true scripts/smoke-test-monitoring.sh`; run completed with `status=Completed`, `totalHits=10000`, `sampledHits=100`, and `findingsCount=1`

## Decisions

- Use the existing `utk-mcp` server as the Kibana integration boundary
- Use the official MCP C# SDK for MCP communication
- Use AlfaGen through an OpenAI-compatible API and keep the model configurable
- Use the official OpenAI .NET SDK for model communication
- Start with daily batch-style monitoring and a manual rerun path instead of real-time streaming
- Start with a server-rendered web UI rather than a SPA
- Use repo-root Skaffold for local development, modeled after `skp-product-change-workflow-service`
- Keep the first local stack limited to app and PostgreSQL only
- Apply EF Core migrations automatically from the app during startup only in explicit local Skaffold mode
- Treat reporting dates as UTC days
- Preserve monitoring run history for reruns
- Expose JSON API endpoints for triggering runs and querying persisted results during development

## Follow Up

- Add scheduled daily execution and integration tests around the monitoring pipeline
