---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-19
service: "[[work/alfa-bank/services/ufr-kpnsb-covenant-monitoring-service|ufr-kpnsb-covenant-monitoring-service]]"
project: [[agents/projects/utk-mcp|UTK MCP]]
ticket:
---

# Kibana Log Analysis: Covenant-Monitoring-Service Errors

## Scope

- Agent project: [[agents/projects/utk-mcp|UTK MCP]]
- Service under investigation: `ufr-kpnsb-covenant-monitoring-service` (internal name: `CovenantMonitoringService`)

## Actions

1. **Searched errors** for `ufr-kpnsb-requestworkflow-service` and `ufr-kpnsb-covenant-monitoring-service`
2. **Traced dependency chain** to `ufr-kpnsb-scoring-service`
3. **Identified root cause**: BKI integration returns 400 on `/CreditHistory/v10`
4. **Named service**: `ufr-kpnsb-covenant-monitoring-service`

## Decisions

- Root issue is not in `covenant-monitoring-service` itself but in upstream `scoring-service`
- Next step: investigate `/CreditHistory/v10` request format or BKI integration

## Handoff

- Covenant monitoring service full name: `ufr-kpnsb-covenant-monitoring-service`
- Error chain: `scoring-service (BKI 400) → covenant-monitoring (ApiException) → requestworkflow (500)`
