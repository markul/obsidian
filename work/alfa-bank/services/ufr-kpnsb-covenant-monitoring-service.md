---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/ufr-kpnsb-covenant-monitoring-service|ufr-kpnsb-covenant-monitoring-service]]"
status: observed
---

# ufr-kpnsb-covenant-monitoring-service

## Repository

- Not recorded in the vault yet.

## Operational Notes

- Observed during Kibana error-chain analysis through `utk-mcp`.
- Internal name recorded in logs: `CovenantMonitoringService`.

## Related Projects

- [[agents/projects/utk-mcp|UTK MCP]]

## Recent Sessions

- [[agents/sessions/2026-04-19-kibana-covenant-errors|2026-04-19 Covenant monitoring errors]]

## Known Issues

- Recorded error chain points through `scoring-service` and BKI `/CreditHistory/v10`; the root issue was not concluded to be inside this service.
