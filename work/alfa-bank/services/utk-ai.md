---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
---

# utk-ai

## Repository

- Local path: `/home/marat/dev/git/alfa/utk-ai`

## Local Development

- `.NET 10` API plus standalone Blazor WebAssembly UI for AI-assisted operational workflows.
- Current first feature is Kibana error monitoring backed by `utk-mcp`, AlfaGen, and PostgreSQL.
- Local development uses Skaffold for PostgreSQL dependencies plus local `dotnet` runs for API and Web.
- PostgreSQL local dependency profile: `skaffold dev -p deps --port-forward`, forwarded on `127.0.0.1:15432`.
- API local run: `cd /home/marat/dev/git/alfa/utk-ai/src/Utk.Ai.Api && dotnet run`, serving `http://localhost:9596`; browser launch is disabled in the API launch profile.
- Web local run: `cd /home/marat/dev/git/alfa/utk-ai/src/Utk.Ai.Web && dotnet run`, serving `http://localhost:9595`.
- Local Docker `utk-mcp` endpoint: `http://127.0.0.1:1985/mcp`.
- API migrations run by default through `Database:ApplyMigrations=true` and can be disabled through appsettings or `Database__ApplyMigrations=false`.
- Current operator preference for this repo is stable `dotnet run`, not hot reload or `dotnet watch`.
- The Web app exposes stored monitoring at `/` and transient Kibana-link investigation at `/plain-search`.

## Related Projects

- [[agents/projects/utk-ai|UTK AI]]
- [[agents/projects/utk-mcp|UTK MCP]]

## Related Services

- [[work/alfa-bank/services/skp-product-change-workflow-service|skp-product-change-workflow-service]]
- [[work/alfa-bank/services/ufr-kpnsb-product-request-workflow-service|ufr-kpnsb-product-request-workflow-service]]

## Known Issues

- Full LLM-trigger smoke path requires `AlfaGen__ApiKey` from local ignored config or Kubernetes secret.
- If Blazor WebAssembly starts returning stale fingerprinted `Utk.Ai.Web*.wasm` 404s after changes, restart only the Web `dotnet run` process to refresh the dev-server static asset map.
