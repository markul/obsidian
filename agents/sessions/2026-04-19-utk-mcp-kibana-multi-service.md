---
tags:
  - agents/session
note-type: session
session-date: 2026-04-19
related-project: [[agents/projects/utk-mcp|UTK MCP]]
related-ticket:
---

# Kibana Log Analysis: Multi-Service Error Chain

## Scope

- Agent project: [[agents/projects/utk-mcp|UTK MCP]]
- Related work: utk-mcp enhancement to support `services` list in `SearchKibanaLogs`

## Actions

1. **Investigated error chain** from `ufr-kpnsb-requestworkflow-service` to `ufr-kpnsb-covenant-monitoring-service` to `ufr-kpnsb-scoring-service`
2. **Identified root cause**: BKI integration failure in `ufr-kpnsb-scoring-service` (POST /CreditHistory/v10 returning 400)
3. **Implemented utk-mcp enhancement**: Updated Kibana log search to accept list of services instead of single service
4. **Added regression test**: Validates whitespace-only input produces no service filter clause

## Decisions

- Service list uses Elasticsearch `bool.should` with OR logic
- Service normalization handles whitespace-only/empty inputs
- Result exposes `Services` as `IReadOnlyList<string>?`

## Validation

- Build: `dotnet build` passed
- Tests: `29/29` passed

## Handoff

- utk-mcp updated to v0.4.0
- MCP tool `search_kibana_logs` now accepts `services: List[str]`
- Can now trace errors across microservice chains in single query
