---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-30
service: "[[work/alfa-bank/services/skp-product-change-workflow-service|skp-product-change-workflow-service]]"
project: "[[agents/projects/utk2-3340|UTK2-3340 Agent Work]]"
ticket: "[[work/alfa-bank/tickets/utk2-3340|UTK2-3340]]"
---

# 2026-04-30 UTK2-3340 CronJob Skaffold

## Goal

- Start implementation for `UTK2-3340`: test cron-job execution on Skaffold by updating the integration-test image to .NET 10 and adding cron-job process support.

## Scope

- Agent project: [[agents/projects/utk2-3340|UTK2-3340 Agent Work]]
- Ticket: [[work/alfa-bank/tickets/utk2-3340|UTK2-3340]]
- Repo: `/home/marat/dev/git/alfa/skp-product-change-workflow-service`
- Branch: `integration-2`

## Actions

- Fetched Jira `UTK2-3340` through `utk-mcp`; ticket is `Develop` and asks to help test cron jobs on Skaffold.
- Retargeted the integration-test project to `net10.0` and aligned direct Microsoft package references with .NET 10 versions.
- Updated `devops/RunTests.groovy` Allure result path from `net8.0` to `net10.0`.
- Reworked `test/Skp.ProductChangeWorkflowService.Tests.Integration/Dockerfile` to use the internal `.NET 10` SDK image, publish the cron job project, copy the cron job output into `/app/cronjob`, install `postgresql16-client`, and include `allurectl`.
- Initially added and then removed a custom `entrypoint.sh`; final image keeps the original `dotnet test` entrypoint.
- Kept the cron job publish output in `/app/cronjob` so integration tests can start cron job processes themselves.
- Validated `dotnet build --no-restore`, `skaffold render --offline=true`, direct integration-test Release build, and cron job publish.
- Investigated Alpine `apk` mirror timeouts. Host curl worked; Docker default build networking timed out; `docker build --network=host` succeeded for both the `skp-docker` SDK image and this repo's integration-test image.
- Confirmed the previous built image had `.NET 10.0.103`, `pg_isready`, `psql`, `allurectl`, and the cron job DLL; after entrypoint rollback, local `dotnet build --no-restore` and `skaffold render --offline=true` still pass.
- Added `devops/wait-for-skaffold-api.sh` after repeated host-run integration test attempts failed early while sidecar containers or forwarded services still appeared unavailable.
- Updated README to document running `./devops/wait-for-skaffold-api.sh` before host-side integration tests after `skaffold dev -p api --port-forward`.
- Ran `skaffold dev -p api --port-forward` for diagnosis. The deployment stabilized in about `1m20s`; there were no pod restarts, OOM signs, disk pressure, or memory pressure. Minikube was around `9 GiB / 32 GiB` memory usage.
- Found stale `skaffold run -p api --port-forward=services` and `kubectl port-forward` processes from earlier runs. They occupied expected local ports and caused Skaffold to create shifted forwards for the new pod.
- Confirmed that after the wait script completed, host-side integration tests reached `localhost:8090` and `localhost:8080` immediately. The run failed two `CheckSebResultStep` assertions where `IsSebCheckSuccess` was `False` instead of expected `True`; it did not fail due to unavailable containers.
- Stopped the live Skaffold run and killed only the exact stale Skaffold/port-forward PIDs. Verified no Skaffold/port-forward processes remained and the expected local ports were free.
- Rechecked from a clean state: Skaffold used the expected local ports `8080`, `8090`, `2000`, `5432`, `9092`, `9000`, and `8088`; deployment stabilized in `1m10s`; readiness script passed in about `8s`.
- Repeated host-side integration tests still failed only two `CheckSebResultStep` assertions with `IsSebCheckSuccess=False`, while `40` tests passed. This confirms the earlier unavailable-container symptoms were caused by stale port-forwards, not cluster resource limits.
- Increased the local Skaffold Kafka sidecar CPU limit from `500m` to `1000m` in `devops/base/k8s-pod.yaml`; verified the rendered manifest shows Kafka at `1000m` while ZooKeeper and Kafka UI stay unchanged.
- Measured startup after the Kafka CPU change: Skaffold rebuilt the service image because the manifest change invalidated the broad Docker context copy, then deployment stabilized in `41.086s`. Host endpoint readiness passed in about `7.5s`. Kafka pod spec confirmed `limits.cpu=1` and there were `0` restarts.
- Doubled Kafka and ZooKeeper caps again for comparison: Kafka `2000m`, ZooKeeper `1000m`. Rendered manifest and live pod spec confirmed `kafka limits.cpu=2` and `zookeeper limits.cpu=1`.
- Measured startup with Kafka `2` CPU / ZooKeeper `1` CPU: deployment stabilized in `30.087s`, endpoint readiness passed in about `7.7s`, and all containers had `0` restarts.
- Increased Kafka again to `4000m` while leaving ZooKeeper at `1000m`. Rendered manifest and live pod spec confirmed `kafka limits.cpu=4` and `zookeeper limits.cpu=1`.
- Measured startup with Kafka `4` CPU / ZooKeeper `1` CPU: deployment stabilized in `23.064s`, endpoint readiness passed in about `7.6s`, and all containers had `0` restarts.
- Rolled back the README and `devops/wait-for-skaffold-api.sh` changes at user request; the local implementation no longer includes that helper or README runbook update.
- Removed local project images and ran `docker builder prune -af`, reclaiming about `16.22GB`, then ran `skaffold dev -p api --port-forward --cache-artifacts=false` for a cold rebuild.
- Cold Skaffold build findings:
  - Service SDK base pull/extract took `138.7s`.
  - Service API restore took about `5.89 min`; the publish steps were under `10s`.
  - Integration-test project restore took about `95s`.
  - Cron job restore inside the integration-test image took `5.57 min`; integration-test build was about `4.5s` and cron job publish about `10.2s`.
  - Thrift mock restore took about `53s`.
- Cold deployment findings:
  - Minikube pulled sidecar dependency images sequentially: WireMock `39.324s`, Postgres `54.833s`, Kafka UI `58.895s`, ZooKeeper `1m44.77s`, Kafka `1m20.126s`.
  - Deployment stabilized in `6m51.075s` with Kafka `limits.cpu=4` and ZooKeeper `limits.cpu=1`.
  - API health returned `Healthy`, WireMock mappings returned `[]`, auth mock token request returned HTTP 200, and all expected ports were forwarded.
  - Stopped Skaffold with `Ctrl+C`; pod deletion completed, no Skaffold or `kubectl port-forward` processes remained, and expected local ports were free.
- Rechecked Docker DNS after the cold Skaffold success:
  - Host/systemd-resolved resolves `binary.alfabank.ru` to `10.212.192.4` through the `snx-tun` split-DNS route for `~alfabank.ru`; `curl` to the Alpine `APKINDEX.tar.gz` returns HTTP 200.
  - Default Docker build/run resolves `binary.alfabank.ru` to `217.12.96.209` using Docker's resolver source from `/run/systemd/resolve/resolv.conf`; direct `wget` to the same `APKINDEX.tar.gz` times out even with a `90s` timeout.
  - `docker build --network=host` resolves `binary.alfabank.ru` to `10.212.192.4` and reaches the same `APKINDEX.tar.gz` with HTTP 200.
  - The minikube container resolves `binary.alfabank.ru` to `10.212.192.4`, so Kubernetes-side dependency image pulls can work even while ordinary Docker build/run containers still have the wrong resolver path.
  - Conclusion: the cold Skaffold run succeeding did not prove Docker default DNS is fixed; default Docker networking remains unreliable for the Alfa Alpine mirror.
- Added the concrete Reqnroll cron job smoke test requested later in the session:
  - `test/Skp.ProductChangeWorkflowService.Tests.Integration/Tests/ProcessPendingProductChangeRequestsCronJob.feature`
  - `test/Skp.ProductChangeWorkflowService.Tests.Integration/Steps/CronJobSteps.cs`
- The scenario runs the real cron job process with `dotnet Skp.ProductChangeWorkflowService.CronJob.dll process-pending-product-change-requests` and asserts only that the exit code is `0`.
- The binding resolves `/app/cronjob` first so it matches the integration-test Kubernetes image; local fallbacks cover `TestContext.CurrentContext.TestDirectory` and the repo cron job `bin/<Configuration>/net10.0` output.
- Validated the new feature/binding with `dotnet build test/Skp.ProductChangeWorkflowService.Tests.Integration/Skp.ProductChangeWorkflowService.Tests.Integration.csproj`; build passed.
- Did not run the scenario locally because the integration stack was not listening on `8080`, `8090`, or `5432`.
- Kubernetes check: `test/Skp.ProductChangeWorkflowService.Tests.Integration/Dockerfile` publishes the cron job to `/app/cronjob`, and `devops/base/k8s-pod.yaml` runs the integration-test container in the same pod as Postgres, Kafka, WireMock, auth mock, and related sidecars, so `localhost` addresses from `appsettings.Local.yaml` should resolve correctly inside the pod.

## Decisions

- Use `postgresql16-client` rather than unversioned `postgresql-client`; the unversioned package was not resolvable from the tested Alpine mirrors, and the local stack uses PostgreSQL 16.
- Keep the test container entrypoint as `dotnet test`; tests should invoke the cron job executable directly from `/app/cronjob`.
- Keep the first cron job smoke test intentionally narrow: launch the process and assert exit code `0`; do not seed or assert database state in this scenario.
- Use `--network=host` for local Docker image validation against Alfa package mirrors.
- Treat local port-forward hygiene as part of the integration-test runbook; stale forwards can make Skaffold bind alternate ports while tests still call the fixed defaults.
- A clean cold Skaffold run is dominated by NuGet restore, base image pulls, and minikube sidecar image pulls; it did not show CPU or memory pressure.
- Do not interpret one successful cold Skaffold build as proof that Docker default DNS is correct; direct no-cache Docker build/run checks still fail without host networking.

## Follow Up

- Decide whether Skaffold local builds need an explicit host-network workaround or whether Docker daemon DNS should be fixed outside the repo so containers use the Alfa/VPN resolver for `*.alfabank.ru`.
- Run the new `ProcessPendingProductChangeRequestsCronJob` scenario inside the Kubernetes integration pod once the local stack is available.
- Investigate the two remaining `CheckSebResultStep` assertion failures separately from Skaffold readiness.
