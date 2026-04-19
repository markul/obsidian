---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-13
related-project: 
related-ticket: 
---

# 2026-04-13 ufr-kpnsb-limit-service-jenkins-1703

## Goal

- Analyze Jenkins build `ufr-kpnsb-limit-service #1703` and identify the real failure point

## Scope

- Repo: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Jenkins build: [ufr-kpnsb-limit-service #1703](https://platform/ci/ufrkpnsb/job/ufr-kpnsb-limit-service/1703/)
- Related notes:
  - [[daily/2026-04-13]]

## Actions

- Read the Alfa internal MCP skill and verified VPN connectivity with `snxctl status`
- Used `utk-mcp` Jenkins tools to fetch build metadata and the full console log for build `#1703`
- Confirmed the build ran commit `2d3cf06f76ef9f31a5cf617efd951d6d874135a5` on branch `integration-1`
- Confirmed the only included change was commit `900cabcc99567fcb8c226765a8c149a83f063a88` touching `test/Ufr.Kpnsb.LimitService.Tests.Integration/Dockerfile`
- Inspected the saved Jenkins console log at `/home/marat/dev/git/alfa/utk-mcp/downloads/jenkins/build-logs/ufr-kpnsb-limit-service/1703/consoleText.log`
- Verified the service image `eco-docker-skp-tmp-snapshots.binary.alfabank.ru/ru.alfabank.integration-test/ufr-kpnsb-limit-service:v0.0.0-integration-1-2026-04-13-11-56-04` was built successfully, saved locally, and pushed successfully to the temporary registry
- Verified the integration-test image was also built and pushed successfully
- Verified unit tests passed (`632` total, `627` passed, `5` skipped) and integration tests passed (`18` total, `17` passed, `1` skipped)
- Verified Allure results uploaded successfully
- Isolated the fatal step in `PublishDockerImageStage`: `docker tag ... No such image`
- Compared with successful build `#1702` to confirm the publish step normally expects a locally available temp image

## Decisions

- Treat the failure as a pipeline or agent-image-locality issue, not an application-code regression
- Treat the Sonar external issues error for missing `MutantTestReports/StrykerReport.json` as non-blocking noise for this build because the pipeline continued past it
- Treat the recent `UTK2-3295` Dockerfile change as validated by the passing integration-test image build and passing integration tests

## Validation

- Jenkins build metadata showed `result: FAILURE`, `branchName: integration-1`, `commitHash: 2d3cf06f76ef9f31a5cf617efd951d6d874135a5`
- Console log evidence:
  - Initial cache pull missed, then local build started:
    - `docker pull ... ufr-kpnsb-limit-service:v0.0.0-integration-1-2026-04-13-11-56-04`
    - `manifest unknown`
  - The same image was later built and named successfully:
    - `#27 naming to eco-docker-skp-tmp-snapshots.binary.alfabank.ru/ru.alfabank.integration-test/ufr-kpnsb-limit-service:v0.0.0-integration-1-2026-04-13-11-56-04 done`
  - The same image was pushed successfully to the temporary registry:
    - `INFO: Successfully pushed docker image: eco-docker-skp-tmp-snapshots.binary.alfabank.ru/ru.alfabank.integration-test/ufr-kpnsb-limit-service:v0.0.0-integration-1-2026-04-13-11-56-04`
  - Final failure:
    - `docker tag eco-docker-skp-tmp-snapshots.binary.alfabank.ru/ru.alfabank.integration-test/ufr-kpnsb-limit-service:v0.0.0-integration-1-2026-04-13-11-56-04 ...`
    - `Error response from daemon: No such image`

## Follow Up

- If this pipeline needs to be hardened, `PublishDockerImageStage` should not assume the temp image still exists locally after test execution; it should either `docker pull` the temp image again before tagging or publish directly from the registry-backed reference
- If the team wants a code change, inspect the shared `modulePipeline` implementation rather than this repository first
