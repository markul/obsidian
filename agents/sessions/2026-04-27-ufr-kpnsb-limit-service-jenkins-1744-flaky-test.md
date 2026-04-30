---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-27
service: "[[work/alfa-bank/services/ufr-kpnsb-limit-service|ufr-kpnsb-limit-service]]"
related-project: "[[agents/projects/utk2-3326|UTK2-3326 Agent Work]]"
related-ticket: "[[work/alfa-bank/tickets/utk2-3326|UTK2-3326]]"
---

# 2026-04-27 ufr-kpnsb-limit-service-jenkins-1744-flaky-test

## Goal

- Investigate failed Jenkins build `ufr-kpnsb-limit-service #1744` and determine why `ProcessLimitGroupAsync_ShouldCreateChangeLimitRequest` is flaky

## Scope

- Repo: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Jenkins build: [ufr-kpnsb-limit-service #1744](https://platform.moscow.alfaintra.net/ci/ufrkpnsb/job/ufr-kpnsb-limit-service/1744/)
- Related notes:
  - [[agents/projects/utk2-3326|UTK2-3326]]
  - [[work/alfa-bank/tickets/utk2-3326|UTK2-3326]]
  - [[agents/sessions/2026-04-24-ufr-kpnsb-limit-service-jenkins-1740]]

## Actions

- Verified Alfa VPN connectivity with `snxctl status`
- Queried Jenkins build metadata and downloaded the `#1744` console log through `utk-mcp`
- Confirmed the merge commit for `#1744` changes only `devops/Dockerfile`
- Isolated the blocking failure to `LimitChangeServiceTests.ProcessLimitGroupAsync_ShouldCreateChangeLimitRequest` with the assertion `Expected value to be 180M, but found 79M`
- Compared `#1744` with earlier feature-branch builds and confirmed the same feature commit had both green and red Jenkins runs before merge
- Reproduced the exact merged commit in a detached local worktree and ran the filtered test successfully
- Re-ran the filtered test set repeatedly and confirmed the failure is not deterministic from the production code path alone
- Traced the flakiness to the unit-test setup:
  - `SingleOpenLimitStateAsync(limitGroup.LmLimitId, ...)` is configured once for the group limit
  - `SingleOpenLimitStateAsync(clientLimit.LmSublimitId, ...)` is configured again for every client sublimit
  - AutoFixture can generate the same integer for `LmLimitId` and one of the `LmSublimitId` values, causing the later mock setup to overwrite the group-limit setup
- Identified a second source of nondeterminism in the same theory: the mocked successful `ChangeLimitAsync` callback used `Fixture.Create<bool>()` to randomly decide whether `SetSubLimitChanged()` runs, even though the production `LimitFacade.ChangeLimitAsync` always sets that flag on the success path
- Patched the test file to normalize duplicate LM identifiers before mock setup and to make the success callback deterministic

## Decisions

- Treat Jenkins build `#1744` as a test-flake investigation, not a runtime regression from the `UTK2-3326` Docker image change
- Fix the nondeterminism in unit tests rather than changing the production `LimitChangeService` logic
- Apply the LM-identifier normalization in the shared `SetupMocksForClientLimits` helper so the same collision pattern does not survive in other tests in the file

## Validation

- `dotnet test test/Ufr.Kpnsb.LimitService.Tests.Unit/Ufr.Kpnsb.LimitService.Tests.Unit.csproj -c Release --filter "FullyQualifiedName~Ufr.Kpnsb.LimitService.Tests.Unit.Application.Services.LimitChangeServiceTests"`
- `for i in $(seq 1 20); do dotnet test test/Ufr.Kpnsb.LimitService.Tests.Unit/Ufr.Kpnsb.LimitService.Tests.Unit.csproj -c Release --no-restore --filter "FullyQualifiedName~ProcessLimitGroupAsync_ShouldCreateChangeLimitRequest"; done`

## Follow Up

- If Jenkins `#1745` or later still fails on the same theory after this patch is applied, inspect other AutoFixture-generated identifier collisions in adjacent tests first
- The separate Sonar `MutantTestReports/StrykerReport.json` error remains a repo-script issue, but it was already present in successful build `#1743` and is not the blocking cause of `#1744`
