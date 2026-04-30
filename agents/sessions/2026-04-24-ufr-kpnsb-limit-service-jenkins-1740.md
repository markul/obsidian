---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-24
service: "[[work/alfa-bank/services/ufr-kpnsb-limit-service|ufr-kpnsb-limit-service]]"
related-project: "[[agents/projects/utk2-3326|UTK2-3326 Agent Work]]"
related-ticket: "[[work/alfa-bank/tickets/utk2-3326|UTK2-3326]]"
---

# 2026-04-24 ufr-kpnsb-limit-service-jenkins-1740

## Goal

- Analyze the latest failed Jenkins build for `ufr-kpnsb-limit-service` and determine whether `UTK2-3326` introduced a real regression

## Scope

- Repo: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Jenkins build: [ufr-kpnsb-limit-service #1740](https://platform.moscow.alfaintra.net/ci/ufrkpnsb/job/ufr-kpnsb-limit-service/1740/)
- Related notes:
  - [[agents/projects/utk2-3326|UTK2-3326]]
  - [[work/alfa-bank/tickets/utk2-3326|UTK2-3326]]
  - [[agents/sessions/2026-04-13-ufr-kpnsb-limit-service-jenkins-1703]]
  - [[daily/2026-04-24]]

## Actions

- Re-read the Alfa internal MCP skill and verified VPN connectivity with `snxctl status`
- Queried Jenkins through `utk-mcp` and identified the red job `ufr-kpnsb-limit-service`
- Fetched job metadata and confirmed the latest completed build is failed build `#1740`
- Fetched build `#1740` metadata and confirmed:
  - branch `feature/UTK2-3326`
  - commit `0c8b109a069187429fd5bf6d20152c443222d019`
  - result `FAILURE`
- Downloaded the full console log to `/home/marat/dev/git/alfa/utk-mcp/downloads/jenkins/build-logs/ufr-kpnsb-limit-service/1740/consoleText.log`
- Searched the console log for the first fatal error and reviewed the final pipeline stages directly
- Re-read [[agents/sessions/2026-04-13-ufr-kpnsb-limit-service-jenkins-1703]] to compare the current failure with the previously documented publish-stage incident

## Decisions

- Treat build `#1740` as a shared pipeline or agent-image-locality failure, not an application-code regression in `UTK2-3326`
- Treat the Debian mirror timeout warnings during image build as non-fatal noise for this run because the image build completed and the pipeline continued
- Treat the `No test is available` lines for `obj/...` assemblies as non-fatal test-discovery noise because the final integration-test summary is green

## Validation

- Jenkins metadata:
  - `ufr-kpnsb-limit-service #1740`
  - branch `feature/UTK2-3326`
  - commit `0c8b109a069187429fd5bf6d20152c443222d019`
- Console log evidence:
  - integration tests passed:
    - `Passed!  - Failed:     0, Passed:    17, Skipped:     1, Total:    18`
  - Allure upload and report generation completed successfully
  - fatal failure happened later in `PublishDockerImageStage`:
    - `docker tag eco-docker-skp-tmp-snapshots.binary.alfabank.ru/ru.alfabank.integration-test/ufr-kpnsb-limit-service:v0.0.0-feature-utk2-3326-2026-04-24-13-12-40 ...`
    - `Error response from daemon: No such image`
- Historical comparison:
  - build `#1703` on `integration-1` failed at the same publish step with the same `docker tag ... No such image` pattern after green tests

## Follow Up

- If the team wants the pipeline fixed, inspect the shared publish stage rather than the service repo first
- The hardening direction remains the same as for build `#1703`: pull the temp image again before tagging, or publish from a registry-backed reference instead of assuming the image is still present locally
