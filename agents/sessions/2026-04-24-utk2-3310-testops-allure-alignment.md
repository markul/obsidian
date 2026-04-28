---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-24
related-project: "[[agents/projects/utk2-3310|UTK2-3310 Agent Work]]"
related-ticket: "[[work/alfa-bank/tickets/utk2-3310|UTK2-3310]]"
---

# 2026-04-24 utk2-3310 testops allure alignment

## Goal

- Make `skp-product-change-workflow-service` send Allure results to TestOps the same way as `ufr-kpnsb-limit-service`

## Scope

- Repo: `/home/marat/dev/git/alfa/skp-product-change-workflow-service`
- Reference repo: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Related notes:
  - [[agents/projects/utk2-3310|UTK2-3310]]
  - [[work/alfa-bank/tickets/utk2-3310|UTK2-3310]]
  - [[agents/sessions/2026-04-22-utk2-3310-jenkins-integration-tests]]
  - [[daily/2026-04-24]]

## Actions

- Re-read the `UTK2-3310` durable notes and then checked the live repo files on `integration-2`
- Compared `devops/Jenkinsfile` and `devops/RunTests.groovy` in `skp-product-change-workflow-service` against `ufr-kpnsb-limit-service`
- Confirmed the Groovy-side Allure/TestOps upload path is already aligned:
  - `runIntegrationTests = true`
  - `integrationTestsPodFile = 'devops/base/k8s-pod.yaml'`
  - `uploadAllureResults()` exists and uses `allurectl upload`
  - results are stashed from `allure-results/**`
- Verified the current `devops/base/k8s-pod.yaml` is a single `Deployment`, so the older multi-document Jenkins parser failure is no longer the immediate blocker for TestOps upload
- Compared the integration-test Dockerfiles and found the remaining mismatch:
  - `limit-service` copies the Alfa TestOps CA certificates into the image and runs `update-ca-certificates`
  - `product-change` downloaded `allurectl` but did not install the internal CA certificates
- Added `cert/test-ops/Alfa-Bank Root CA 2012.cer` and `cert/test-ops/Alfa-Bank Sub2 CA 2012.cer` to `skp-product-change-workflow-service`
- Updated `test/Skp.ProductChangeWorkflowService.Tests.Integration/Dockerfile` to:
  - copy both certs into `/usr/local/share/ca-certificates/`
  - set readable permissions on them
  - run `update-ca-certificates`
  - keep the existing `allurectl` and `init-db.sh` executable setup
- Started a local `docker build -f test/Skp.ProductChangeWorkflowService.Tests.Integration/Dockerfile ...` verification run
- After pushing the change to Bitbucket, monitored Jenkins for branch `feature/UTK2-3310` and confirmed build `skp-product-change-workflow-service #270` ran commit `7a2d5121325f86afa13a7efeab108477b674f745`
- Downloaded and reviewed the Jenkins console log for build `#270`

## Decisions

- Treat the missing TestOps CA import as the material difference preventing product-change from being configured the same way as limit-service for Allure upload
- Keep the same repo-local certificate pattern as limit-service instead of relying on the base image trust store
- Treat the current Docker verification as partial but meaningful because the build passed the new cert-path and Dockerfile-parse path and only stalled later in the existing network download layer

## Validation

- File comparison:
  - `devops/RunTests.groovy` already contains the same `uploadAllureResults()` logic shape as `limit-service`
  - `devops/Jenkinsfile` already enables integration tests
- Dockerfile delta applied:
  - added `COPY` lines for the two TestOps certs
  - added `update-ca-certificates`
- Local build verification:
  - Docker accepted the new Dockerfile and new cert paths
  - the build reached the existing `apt-get/curl` network step in the final stage
  - the run then stalled on the pre-existing internal network download path for `allurectl`, so there was no full image completion in this pass
- Jenkins build `#270` validation:
  - build commit matched the pushed change: `7a2d5121325f86afa13a7efeab108477b674f745`
  - the updated cert-copy steps were executed in Jenkins:
    - `COPY [./cert/test-ops/Alfa-Bank Root CA 2012.cer, ...]`
    - `COPY [./cert/test-ops/Alfa-Bank Sub2 CA 2012.cer, ...]`
    - `RUN chmod 644 ... && update-ca-certificates ...`
  - Jenkins showed certificate update execution succeeded:
    - `Updating certificates in /etc/ssl/certs...`
  - the fatal failure happened later and was unrelated to the TestOps CA change:
    - `Could not push image: An image does not exist locally with the tag: eco-docker-skp-tmp-snapshots.../skp-product-change-workflow-service`
  - unit tests were green before that failure:
    - `Passed!  - Failed: 0, Passed: 216, Skipped: 0`
  - Sonar also emitted a non-blocking error for missing `/app/aspnetapp/MutantTestReports/StrykerReport.json`, but the pipeline continued past it

## Follow Up

- The pushed change is now validated as part of the Jenkins image build path; the remaining red build is a later pipeline image-push problem rather than a cert/trust-store problem
- If the team wants the branch green, the next fix should target the shared Jenkins or Artifactory docker-push stage that loses the main service image before push
