---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-05
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-05 UTK AI Aggregation Persistence

## Goal

- Persist MCP aggregation buckets and their representative examples separately from AlfaGen findings so aggregated monitoring output can be inspected directly.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[daily/2026-05-05]]
  - [[agents/sessions/2026-05-05-utk-mcp-kibana-aggregation|2026-05-05 UTK MCP Kibana Aggregation]]

## Actions

- Added domain entities for `MonitoringAggregationBucket` and `MonitoringAggregationExample`.
- Extended EF Core mapping and generated migration `20260505084606_AddMonitoringAggregationBuckets`.
- Changed the aggregated MCP client path to return both flattened hits for existing AlfaGen analysis and structured bucket metadata with examples.
- Updated `MonitoringRunProcessor` to store bucket keys, key parts JSON, document counts, and all representative examples before running AlfaGen.
- Added `GET /api/monitoring-runs/{runId}/aggregation-buckets` for API inspection of persisted buckets and examples.
- Restarted the local API watcher after `dotnet watch` hot reload crashed; startup applied the new migration to the local PostgreSQL database.
- Follow-up: changed aggregated monitoring grouping from `app_id` only to `app_id` plus `sourceContext`.
- Follow-up: changed aggregated AlfaGen mapping so persisted finding `occurrences` are derived from the matched aggregation bucket `document_count` instead of AlfaGen's guessed count or the number of representative examples.
- Follow-up: changed aggregated monitoring grouping to `app_id`, `sourceContext`, and synthetic MCP field `utk.normalizedEndpoint` after `sourceContext` alone mixed multiple outbound dependencies in `LoggingDelegatingHandler`.
- Added same-origin post-processing in AlfaGen result mapping so HTTP findings with the same normalized dependency endpoint merge even when symptoms differ, such as `Connection reset by peer` and HTTP `503`.
- Capped long `message` and `stackTrace` fields in the AlfaGen prompt while keeping full evidence persisted, after `skp-product-change-workflow-service` aggregated analysis returned provider HTTP `500`.
- Added an aggregated-strategy fallback that persists bucket-derived findings when AlfaGen fails, so MCP aggregation results are not lost.

## Decisions

- Keep existing finding/evidence persistence unchanged.
- Store aggregation output as run-scoped raw MCP grouping data, not as LLM findings, so raw and aggregated strategies remain comparable.
- Keep occurrence counts tied to real MCP bucket `document_count` values and sum distinct matched buckets when multiple symptoms map to the same origin.
- Keep full evidence values in PostgreSQL; prompt truncation only affects the AlfaGen request payload.

## Validation

- `dotnet build Utk.Ai.slnx`
- `curl -sf http://localhost:9596/health`
- `GET /api/monitoring-runs/{runId}/aggregation-buckets` returns `[]` for pre-migration runs.
- Local Postgres contains `monitoring_aggregation_buckets` and `monitoring_aggregation_examples`.
- Verification run `cb96d75d-e07d-44ef-b1e0-1a6875b5bb4c` completed with buckets `343`, `20`, `17`, and `1`, and finding occurrences persisted from those bucket hit counts.
- Verification run `c8621f2f-16d3-4911-a8d4-2725f836b2e1` completed with `381` total hits, five persisted buckets, and findings `LimitStateOData=346`, `CalendarService=34`, `DomainException=1`.
- Verification run `51a41722-1c12-4b0f-a8ec-f078bf82b3d8` for `skp-product-change-workflow-service` completed through AlfaGen with `47,109` total hits and two findings: Kafka SASL auth failures `47,108` and `skp-deal-service` HTTP connection failure `1`.

## Current State

- Local stack is running: Skaffold deps on PostgreSQL `15432`, API watcher on `9596`, Web watcher on `9595`, and local Docker `utk-mcp` on `1985`.
- `LimitStateOData` connection-reset and HTTP `503` errors are now stored as one finding because they share the same dependency origin.
- `skp-product-change-workflow-service` aggregated monitoring no longer leaves the toast in error for the May 4, 2026 data set; prompt truncation lets AlfaGen analyze the two aggregated buckets.
