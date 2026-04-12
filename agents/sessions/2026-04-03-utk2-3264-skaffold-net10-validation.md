# UTK2-3264 skaffold .NET 10 validation

## Scope

- Ticket: [[work/alfa-bank/tickets/active/utk2-3264|UTK2-3264]]
- Project: [[agents/projects/utk2-3264|UTK2-3264 agent project]]
- Repository: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Branch: `feature/UTK2-skaffold-wiremock-net10-v2`

## Goal

- Verify that the latest local branch state, which already includes `integration-1`, still works with the skaffold + WireMock setup after the `.NET 10` changes

## Actions

- Verified `snxctl status` before Alfa work and paused work while the corporate VPN was disconnected
- Compared `feature/UTK2-skaffold-wiremock-net10-v2` against `integration-1` and confirmed the feature branch is ahead while `integration-1` has no extra commits
- Revalidated the local toolchain state: `minikube` running, `.NET SDK 10.0.105` installed, `skaffold v2.18.2`, Docker available
- Confirmed `dotnet build Ufr.Kpnsb.LimitService.sln` succeeds on the current branch
- Confirmed unit coverage stays green with `625 passed, 5 skipped, 0 failed`
- Ran `skaffold run -p api` with VPN enabled and confirmed the deployment rolled out successfully
- Verified the local forwarded stack with `http://127.0.0.1:8080/health` and `http://127.0.0.1:2000/__admin/mappings`
- Ran the integration suite against the live skaffold deployment and confirmed the WireMock flow still passes
- Cleaned up the local deployment with `skaffold delete -p api`

## Decisions

- Treat corporate VPN connectivity as a hard precondition for this ticket because the local skaffold flow depends on internal registries and feeds
- Keep `feature/UTK2-skaffold-wiremock-net10-v2` as the validation branch; there is no missing `integration-1` delta to merge locally first

## Validation

- `snxctl status`
- `git rev-list --left-right --count feature/UTK2-skaffold-wiremock-net10-v2...integration-1`
- `dotnet build Ufr.Kpnsb.LimitService.sln`
- `dotnet test test/Ufr.Kpnsb.LimitService.Tests.Unit/Ufr.Kpnsb.LimitService.Tests.Unit.csproj --no-build`
- `skaffold run -p api`
- `kubectl rollout status deployment/ufr-kpnsb-limit-service --timeout=120s`
- `curl -fsS http://127.0.0.1:8080/health`
- `curl -fsS http://127.0.0.1:2000/__admin/mappings`
- `dotnet test test/Ufr.Kpnsb.LimitService.Tests.Integration/Ufr.Kpnsb.LimitService.Tests.Integration.csproj --no-build`
- `skaffold delete -p api`

## Outcome

- Current local state is good: the branch already contains `integration-1`, the `.NET 10` solution build succeeds, the skaffold deployment comes up, and the integration suite finishes with `12 passed, 1 skipped, 0 failed`
