---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-28
service: "[[work/alfa-bank/services/skp-product-change-workflow-service|skp-product-change-workflow-service]]"
related-project: "[[agents/projects/utk2-3329|UTK2-3329 Agent Work]]"
related-ticket: "[[work/alfa-bank/tickets/utk2-3329|UTK2-3329]]"
---

# 2026-04-28 utk2-3329-outbound-header-filter

## Goal

- Remove outbound HTTP header logging from `UTK2-3329` while preserving the useful request and response transport fields

## Scope

- Repo: `/home/marat/dev/git/alfa/skp-product-change-workflow-service`
- Jira: [UTK2-3329](https://jira.moscow.alfaintra.net/browse/UTK2-3329)
- Related notes:
  - [[agents/projects/utk2-3329|UTK2-3329]]
  - [[work/alfa-bank/tickets/utk2-3329|UTK2-3329]]
  - [[daily/2026-04-28]]

## Actions

- Inspected the current outbound logging wiring and confirmed the packaged `CustomLoggingHttpMessageHandler` always enriches `httpHeaders`
- Replaced the package handler registration in `src/Skp.ProductChangeWorkflowService.Infrastructure/Extensions/ServiceCollectionExtensions.cs` with a local `OutboundHttpLoggingDelegatingHandler`
- Added `src/Skp.ProductChangeWorkflowService.Infrastructure/Extensions/OutboundHttpLoggingDelegatingHandler.cs` to log method, URI, status code, elapsed time, and optional request or response body fields without logging headers
- Added `src/Skp.ProductChangeWorkflowService.Infrastructure/Options/HttpPayloadLoggingOptions.cs` and bound `HttpPayloadLogging.MaxLoggedBodyLength` from config so logged outbound bodies are truncated to a configured limit
- Added `test/Skp.ProductChangeWorkflowService.Tests.Unit/Infrastructure/Extensions/OutboundHttpLoggingDelegatingHandlerTests.cs` with coverage for normal response logging, exception logging, and payload truncation
- Kept the outbound logger category as `AlfaBank.Http.Message` to preserve the existing search surface in log tooling
- Split the new handler into its own file after `SA1402` rejected multiple types in `ServiceCollectionExtensions.cs`
- Rebuilt the solution and reran the unit-test project after the handler swap
- Checked the current shell and cluster state and found no running `skaffold` process or live pod in the active kube context, so fresh runtime verification of the new outbound log shape is still pending

## Decisions

- Replace the package outbound handler rather than trying to suppress individual header fields, because the package handler does not expose a header toggle
- Preserve the `AlfaBank.Http.Message` logger category so the change only removes the dangerous `httpHeaders` enrichment instead of forcing downstream query changes
- Name the truncation option `MaxLoggedBodyLength` instead of `BodyLogSizeLimit` so the setting reads as “how much body text is written to logs,” not a generic HTTP size limit

## Validation

- `dotnet build -m:1`
- `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Unit/Skp.ProductChangeWorkflowService.Tests.Unit.csproj -m:1`

## Follow Up

- Start `skaffold` again and confirm a live outbound call produces `AlfaBank.Http.Message` entries without `httpHeaders`
