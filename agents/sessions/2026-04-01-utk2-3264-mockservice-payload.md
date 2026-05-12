---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-01
service: "[[work/alfa-bank/services/ufr-kpnsb-limit-service|ufr-kpnsb-limit-service]]"
project: "[[agents/projects/utk2-3264|UTK2-3264 agent project]]"
ticket: "[[work/alfa-bank/tickets/utk2-3264|UTK2-3264]]"
---

# UTK2-3264 MockService to WireMock migration

## Scope

- Ticket: [[work/alfa-bank/tickets/utk2-3264|UTK2-3264]]
- Agent project: [[agents/projects/utk2-3264|UTK2-3264 agent project]]
- Repository: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`

## Goal

- Replace the custom `test/Ufr.Kpnsb.Tests.MockService` sidecar with WireMock for the local skaffold flow and migrate integration fixtures and steps to the WireMock admin API

## Actions

- Replaced the mock sidecar in `devops/base/k8s-pod.yaml` with `docker-hub.binary.alfabank.ru/sheyenrath/wiremock.net:1.7.4`
- Updated `devops/base/k8s-service.yaml`, `skaffold.yaml`, `devops/Dockerfile`, and `Ufr.Kpnsb.LimitService.sln` to remove the custom MockService build/runtime path
- Added WireMock integration-step support that loads mapping files from a folder and clears mappings through `IWireMockAdminApi`
- Migrated integration fixture files from the old flat mock contract to explicit `request` and `response` objects
- Updated integration feature files to use `я настраиваю WireMock из файлов ...` and `я очищаю WireMock`
- Removed the obsolete `test/Ufr.Kpnsb.Tests.MockService` source tree from the repository

## Decisions

- Use WireMock as the standard mock sidecar for the local skaffold environment instead of maintaining a custom mock service
- Keep fixture matching primarily on HTTP method, path, and query parameters; support request-body matching in the new fixture contract when a scenario needs it
- Convert fixtures in place to keep the current scenario structure intact and avoid introducing a second mock-data convention

## Validation

- `dotnet build test/Ufr.Kpnsb.LimitService.Tests.Integration/Ufr.Kpnsb.LimitService.Tests.Integration.csproj`
- `dotnet test test/Ufr.Kpnsb.LimitService.Tests.Integration/Ufr.Kpnsb.LimitService.Tests.Integration.csproj --no-build`
- `skaffold run -p api`

## Follow Up

- `src/Ufr.Kpnsb.LimitService.Api/appsettings.Local.yaml` was updated from `localhost:9000` to `localhost:2000`, and the live pod was rebuilt with that config
- After the port fix, integration failures moved from infrastructure errors to WireMock request-matching problems for OData URLs in `001_GetGuarantorsAsync`
- Stabilized the local stack by stopping the lingering `skaffold dev` watcher and using an explicit `kubectl port-forward` for `8080`, `2000`, and `5432`
- Fixed request-id lookup failures by normalizing query-bearing fixture URLs before building WireMock regex matchers, which removed `%20` vs space mismatches in OData requests
- Fixed asynchronous `002_AddAsync_LimitGroups` processing by widening the `{datetime}` matcher to accept URL-encoded timestamps with fractional seconds, which restored the `SubjectRisksWsrm` mock and let `client_limit.is_checked` reach `true`
- Verified the end state with `dotnet test test/Ufr.Kpnsb.LimitService.Tests.Integration/Ufr.Kpnsb.LimitService.Tests.Integration.csproj --no-build`, which finished with `Passed: 11, Skipped: 1, Failed: 0`
