---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-28
service: "[[work/alfa-bank/services/skp-product-change-workflow-service|skp-product-change-workflow-service]]"
project:
ticket:
---

# 2026-04-28 skp-product-change-workflow-service-integration-tests

## Goal

- Run the integration tests in `skp-product-change-workflow-service`

## Scope

- Repo: `/home/marat/dev/git/alfa/skp-product-change-workflow-service`
- Solution: `Skp.ProductChangeWorkflowService.sln`
- Integration tests: `test/Skp.ProductChangeWorkflowService.Tests.Integration/Skp.ProductChangeWorkflowService.Tests.Integration.csproj`

## Actions

- Read the repo-local `AGENTS.md` and confirmed the intended integration command is `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Integration/Skp.ProductChangeWorkflowService.Tests.Integration.csproj`
- Read the integration test project and confirmed it is meant to be run directly rather than via solution-level `dotnet test`
- Ran the integration test project from the repo root
- Inspected the integration test `appsettings.json` after failure and confirmed the local dependency endpoints are:
  - API: `http://localhost:8080`
  - WireMock: `http://localhost:2000`
  - auth mock / token endpoint: `http://localhost:8090/connect/token`
  - PostgreSQL: `localhost:5432`
  - Kafka: `localhost:9092`
- Checked local listeners and confirmed none of the expected local ports were listening during the run
- Read `skaffold.yaml` and confirmed the `api` profile is the documented way to bring up both dependencies and the service API with local forwards for `8080`, `8090`, `2000`, `5432`, and `9092`
- Verified local prerequisites before startup: `minikube` was already running, `kubectl` was pointed at the `minikube` context, and `skaffold v2.18.2` was installed
- Started `skaffold dev -p api --port-forward` from the repo root and waited for the deployment to stabilize
- Confirmed the forwarded local endpoints were live:
  - `GET http://127.0.0.1:8080/health` -> `Healthy`
  - `POST http://127.0.0.1:8090/connect/token` -> `200`
  - local listeners appeared on `8080`, `8090`, `2000`, `5432`, and `9092`
- Re-ran the same integration test command against the live local stack
- Stopped the long-running Skaffold session cleanly with `Ctrl+C`, which deleted the temporary deployment and service from the `default` namespace
- Updated the repo-local `AGENTS.md` to add an explicit integration-test runbook with Skaffold startup, readiness checks, the direct `dotnet test` command, and the common `localhost:8090` failure interpretation
- Refined the same `AGENTS.md` guidance to distinguish Skaffold modes: recommend `skaffold run -p api --port-forward=services` for one-off integration passes and keep `skaffold dev -p api --port-forward` for iterative work
- Tightened the same `AGENTS.md` guidance again so `skaffold run` explicitly uses `--port-forward=services` and does not rely on the bare `--port-forward` form
- Corrected the cleanup guidance in `AGENTS.md` again after verifying local Skaffold support for `delete`: use `skaffold delete -p api` after `skaffold run -p api --port-forward=services` and keep `pkill -f "skaffold run"` explicitly disallowed
- Narrowed the same `AGENTS.md` runbook so the integration-test instructions are explicitly `api`-profile-only and no longer mention dependency-only Skaffold flow
- Restructured `AGENTS.md` again to give Skaffold its own dedicated section, move general Skaffold usage out of `Testing Guidelines`, and leave the integration-test instructions as a short `api`-profile-specific flow that points back to the dedicated Skaffold section

## Findings

- The test run failed immediately during scenario setup, before business assertions
- All `42` integration tests failed
- The common failure was `System.Net.Http.HttpRequestException: Connection refused (localhost:8090)`
- The failing call happens in the authorization setup hook when tests request a token from `http://localhost:8090/connect/token`
- Because the authorization hook failed, each scenario’s actual Given/When/Then steps were skipped after the setup error
- The repo `README.md` documents that local integration dependencies should be brought up with Skaffold and explicitly lists `localhost:8090` as the auth mock port
- After the Skaffold-backed local environment was started, the integration suite passed cleanly
- Final passing result: `Passed: 42, Failed: 0, Skipped: 0`

## Recommended Automation Workflow

For a one-off automated integration pass, keep the flow aligned with the repo runbook and use the `api` profile only:

1. Deploy: `skaffold run -p api --port-forward=services`
2. Wait for `http://127.0.0.1:8080/health` and `http://127.0.0.1:8090/connect/token`
3. Run tests: `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Integration/Skp.ProductChangeWorkflowService.Tests.Integration.csproj`
4. Cleanup: `skaffold delete -p api`

## Validation

- `dotnet test test/Skp.ProductChangeWorkflowService.Tests.Integration/Skp.ProductChangeWorkflowService.Tests.Integration.csproj`
- `ss -ltn '( sport = :8090 or sport = :8080 or sport = :2000 or sport = :5432 or sport = :9092 )'`
- `curl -fsS http://127.0.0.1:8080/health`
- `curl -X POST http://127.0.0.1:8090/connect/token -H "Content-Type: application/x-www-form-urlencoded" -d "grant_type=client_credentials&client_id=skp-product-change-workflow-service&client_secret=skp-product-change-workflow-service"`

## Outcome

- The first direct test run correctly exposed a missing-environment issue rather than an application failure
- After bringing up the local environment, the integration suite passed cleanly: `Passed: 42, Failed: 0, Skipped: 0`
- The repo-local integration-test instructions should stay `api`-profile-only; dependency-only Skaffold modes belong to broader local-development docs, not to the integration-test runbook
- The repo-local guidance now reflects that split structurally: general Skaffold work lives in a dedicated section, while testing keeps only the short integration-test path
