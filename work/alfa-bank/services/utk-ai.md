---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
status: active
---

# utk-ai

## Repository

- Local path: `/home/marat/dev/git/alfa/utk-ai`

## Local Development

- `.NET 10` API plus standalone Blazor WebAssembly UI for AI-assisted operational workflows.
- Current first feature is Kibana error monitoring backed by `utk-mcp`, AlfaGen, and PostgreSQL.
- Local development uses Skaffold for PostgreSQL dependencies plus local `dotnet` runs for API and Web.

## Related Projects

- [[agents/projects/utk-ai|UTK AI]]
- [[agents/projects/utk-mcp|UTK MCP]]

## Related Services

- [[work/alfa-bank/services/skp-product-change-workflow-service|skp-product-change-workflow-service]]
- [[work/alfa-bank/services/ufr-kpnsb-product-request-workflow-service|ufr-kpnsb-product-request-workflow-service]]

## Recent Sessions

- [[agents/sessions/2026-04-28-utk-ai-bootstrap|2026-04-28 bootstrap]]
- [[agents/sessions/2026-04-28-utk-ai-initial-plan|2026-04-28 initial plan]]
- [[agents/sessions/2026-04-29-utk-ai-monitoring-prototype|2026-04-29 monitoring prototype]]

## Known Issues

- Full LLM-trigger smoke path requires `AlfaGen__ApiKey` from local ignored config or Kubernetes secret.
