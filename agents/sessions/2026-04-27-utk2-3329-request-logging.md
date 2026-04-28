---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-27
related-project: "[[agents/projects/utk2-3329|UTK2-3329 Agent Work]]"
related-ticket: "[[work/alfa-bank/tickets/utk2-3329|UTK2-3329]]"
---

# 2026-04-27 utk2-3329-request-logging

## Goal

- Start `UTK2-3329` and implement the first pass of inbound and outbound request logging for `skp-product-change-workflow-service`

## Scope

- Repo: `/home/marat/dev/git/alfa/skp-product-change-workflow-service`
- Jira: [UTK2-3329](https://jira.moscow.alfaintra.net/browse/UTK2-3329)
- Related notes:
  - [[agents/projects/utk2-3329|UTK2-3329]]
  - [[work/alfa-bank/tickets/utk2-3329|UTK2-3329]]
  - [[daily/2026-04-27]]

## Actions

- Verified Alfa VPN connectivity with `snxctl status`
- Queried Jira issue `UTK2-3329` through `utk-mcp` and confirmed the request is to add adequate inbound and outbound request logging for this service
- Inspected the local API and infrastructure wiring and confirmed inbound request middleware already existed through `UseRequestLogging()`, while outbound typed and OData client registration had no explicit logging attachment
- Decompiled the cached `Skp.Logging`, `Skp.HttpClient`, and `Skp.OData.Client` packages to confirm the available logging hooks and handler-order behavior
- Enabled detailed inbound HTTP logging in `src/Skp.ProductChangeWorkflowService.Api/Logging/LoggingExtensions.cs`
- Added `LogBody(IgnoreLogBodyOptions.IgnoreResponse)` to the public POST actions in `StepController`, `EventController`, and `ProcessController`
- Enabled outbound client logging centrally in `src/Skp.ProductChangeWorkflowService.Infrastructure/Extensions/ServiceCollectionExtensions.cs` for typed HTTP clients, OData clients, and SKP bearer-auth clients
- Added unit coverage in `test/Skp.ProductChangeWorkflowService.Tests.Unit/Api/ApplicationAvailabilityTests.cs` for the detailed HTTP logging option, outbound logging handler registration, and endpoint body-logging attributes
- Fixed one follow-up compile issue by importing `Microsoft.OData.Client` for the new OData helper method
- Compared the live skaffold pod logs and confirmed that inbound body logging was still missing even on `LogBody(...)` actions
- Identified the root cause as middleware ordering: `UseRequestLogging()` was running before `UseRouting()`, so endpoint metadata was unavailable when the logging middleware checked for `LogBodyAttribute`
- Moved `UseRequestLogging()` after `UseRouting()` and revalidated the solution with green build and unit tests
- Narrowed `LogBody(IgnoreLogBodyOptions.IgnoreResponse)` back to `StepController.RunAsync` only, because that endpoint can return the large workflow context string while the process and event endpoints return much smaller payloads

## Decisions

- Use the packaged `Skp.Logging` HTTP message handler instead of adding a repo-local custom outbound handler for the first pass
- Log inbound request bodies but skip inbound response bodies on the workflow endpoints to avoid duplicating large workflow context payloads in logs
- Place the outbound logging handler before bearer-auth handlers so transport logs do not record generated bearer tokens
- Keep request logging before auth but after routing, which preserves unauthenticated-request visibility while allowing endpoint metadata-based body logging

## Validation

- `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Unit/Skp.ProductChangeWorkflowService.Tests.Unit.csproj --filter ApplicationAvailabilityTests`
- `dotnet build -m:1`
- `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Unit/Skp.ProductChangeWorkflowService.Tests.Unit.csproj -m:1`

## Follow Up

- Exercise one real local request path that reaches a downstream service and inspect the resulting log entries for noise, field usefulness, and body size
- If the package outbound logging is too verbose in practice, replace it with a filtered or truncated handler in a follow-up pass before delivery
