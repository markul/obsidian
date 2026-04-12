# 2026-04-02 utk-mcp-jenkins-commit-hash

## Goal

- Add top-level commit, branch, and branch URL fields to Jenkins build responses in `utk-mcp`

## Scope

- Agent project: [[agents/projects/utk-mcp|UTK MCP]]
- Repo: `/home/marat/dev/git/alfa/utk-mcp`

## Actions

- Read the current Jenkins tool, model, client, and tests in `utk-mcp`
- Added `CommitHash` to `JenkinsBuildResult`
- Added `BranchName` to `JenkinsBuildResult`
- Added `BranchUrl` to `JenkinsBuildResult`
- Mapped `CommitHash` from Jenkins Git `lastBuiltRevision.SHA1`, with changeset `commitId` as fallback
- Mapped `BranchName` from Jenkins Git `lastBuiltRevision.branch[].name`, with `buildsByBranchName` as fallback
- Derived `BranchUrl` from the preferred Jenkins Git remote plus the configured Bitbucket base URL
- Normalized Jenkins branch names so values like `refs/remotes/origin/feature/...` become `feature/...` and `origin/dev` becomes `dev`
- Extended the Jenkins build-list query to request `commitId` as well as commit message
- Fixed the Jenkins `tree=` selector after live validation showed `buildsByBranchName[*][buildNumber]` returns HTTP `500` on the target Jenkins; switched to `buildsByBranchName[buildNumber]`
- Updated unit tests to cover the new top-level field for single-build and build-list responses
- Ran `dotnet test test/UtkMcp.Tests.Unit/UtkMcp.Tests.Unit.csproj --filter JenkinsClientTests`
- Ran `dotnet test test/UtkMcp.Tests.Unit/UtkMcp.Tests.Unit.csproj`
- Rebuilt local Docker with `docker compose up -d --build`
- Verified `get_jenkins_job_builds(serviceName=\"ufr-kpnsb-limit-service\", limit=5)` against the local MCP endpoint and confirmed recent builds now expose both `commitHash` and `branchName`

## Decisions

- Expose a single top-level `CommitHash` while keeping the full `Changes` array for multi-commit detail
- Expose a single top-level `BranchName` alongside `CommitHash`
- Expose a top-level `BranchUrl` that points at the Bitbucket repo browser for the normalized branch
- Prefer Jenkins Git `BuildData.lastBuiltRevision` as the primary source for top-level SCM metadata because it stays populated when `changeSets` is empty
- Prefer the service repository `BuildData` action over shared pipeline-repo metadata when multiple Git actions are present

## Validation

- Jenkins client tests passed: `8` passed
- Full unit test project passed: `17` passed
- Live MCP verification returned branch examples:
  - `#1671` -> `feature/UTK2-skaffold-wiremock-net10-v2`
  - `#1670` -> `feature/UTK2-skaffold-wiremock-automapper-fix-net10`
  - `#1667` -> `dev`
- Live MCP verification returned a branch URL example:
  - `#1671` -> `https://git.moscow.alfaintra.net/projects/UFRKPNSB/repos/ufr-kpnsb-limit-service/browse?at=refs%2Fheads%2Ffeature%2FUTK2-skaffold-wiremock-net10-v2`

## Follow Up

- If needed later, decide whether Jenkins should also expose the raw unnormalized branch ref alongside the user-facing `branchName`
- If needed later, decide whether Jenkins should expose repository coordinates separately instead of only the derived `branchUrl`
