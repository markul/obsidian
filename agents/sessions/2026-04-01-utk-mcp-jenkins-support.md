---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-01
service: "[[work/alfa-bank/services/utk-mcp|utk-mcp]]"
project: "[[agents/projects/utk-mcp|UTK MCP]]"
ticket: "[[work/alfa-bank/tickets/utk2-3243|UTK2-3243]]"
---

# 2026-04-01 utk-mcp-jenkins-support

## Goal

- Start work on Jenkins support for `utk-mcp`

## Scope

- Agent project: [[agents/projects/utk-mcp|UTK MCP]]
- Repo: `/home/marat/dev/git/alfa/utk-mcp`
- Related notes:
  - [[daily/2026-04-01]]
  - [[work/alfa-bank/tickets/utk2-3243|UTK2-3243]]

## Actions

- Declared Jenkins support as the next active capability to add to `utk-mcp`
- Created today’s daily note and this session note for the workstream
- Updated the `utk-mcp` agent project to track Jenkins support as current focus
- Inspected the current provider wiring and confirmed Jenkins was registered but still status-only
- Added Jenkins job and build models, provider interface methods, MCP tools, and client implementation
- Added Jenkins unit tests and fixed stale test doubles after the interface expansion
- Verified the repo with `dotnet test UtkMcp.sln -v minimal`
- Checked `https://platform/ci/ufrkpnsb/api/json?pretty=true` over VPN to inspect the live Jenkins REST shape
- Used the saved `/home/marat/dev/git/alfa/utk-mcp/jenkins-api.json` payload to confirm the root folder response shape
- Added Jenkins overview models, client method, MCP tool, and tests for the configured root folder endpoint
- Re-ran `dotnet test UtkMcp.sln -v minimal` after the overview change
- Configured live Jenkins credentials in local `.env` and validated overview, job, and build requests against `https://platform/ci/ufrkpnsb/`
- Rebuilt and redeployed the local `utk-mcp` Docker service so the running MCP endpoint picked up the new Jenkins tools
- Recreated the Docker container with `UTK_MCP_Jenkins__SslVerify=false` so live Jenkins tool calls could succeed inside the container despite the internal CA chain
- Added flattened `changes` support to `get_jenkins_build` from Jenkins `changeSets` data and redeployed the service
- Added `get_bitbucket_branch` to return branch identity plus build metadata for the branch's latest commit
- Added `buildId` to Bitbucket branch build entries by extracting the trailing numeric segment from the Jenkins redirect URL
- Split Jenkins config into `UTK_MCP_Jenkins__BaseUrl` for the host and `UTK_MCP_Jenkins__OrgProject` for the scoped folder path, then rebuilt and redeployed the MCP server
- Added `get_jenkins_build_log` to return Jenkins `consoleText` with line count and canonical URLs for a build
- Added `get_jenkins_job_builds(serviceName, dateFrom?, dateTo?, limit)` using Jenkins `tree=builds{0,N}` plus client-side timestamp filtering
- Extended `get_jenkins_job_builds` so each returned build includes `changesSummary` derived from Jenkins `changeSets`
- Extended `get_jira_issue` with optional `customFields`; requested Jira custom fields can now be resolved by field key or display name and returned in the response
- Added `get_bitbucket_compare_diff` to accept incoming Bitbucket compare links and return structured compare diff data

## Decisions

- Jenkins support should be tracked as `utk-mcp` repo work, not as a separate agent project yet
- Session continuity for this work should live in `agents/sessions/` and link back to [[agents/projects/utk-mcp|UTK MCP]]
- The first Jenkins slice should stay read-only and cover job/build lookup before broader pipeline or console output support
- Slash-delimited job paths such as `folder/job` should be accepted by MCP and translated inside the client to Jenkins `/job/.../job/...` paths
- The Jenkins base path in this environment is under `/ci/ufrkpnsb/`, and the root `/api/json` endpoint is authenticated
- The configured Jenkins base URL should be treated as a first-class MCP resource because its root `api/json` returns useful folder metadata and visible jobs
- Basic auth with the configured username form works for this Jenkins, and read-only `GET` requests do not require crumbs
- Jenkins requests should compose `BaseUrl + OrgProject + relative API path` instead of storing the scoped folder directly in `BaseUrl`
- Build listing should keep the MCP input aligned with service naming while still translating to Jenkins job paths internally

## Validation

- `dotnet test UtkMcp.sln -v minimal` passed
- The saved `jenkins-api.json` payload is a root folder response with 38 jobs, `useCrumbs=false`, and `useSecurity=true`
- Anonymous access to `https://platform/ci/ufrkpnsb/api/json?pretty=true` returns `403 Forbidden` and redirects to `/securityRealm/commenceLogin`
- Response headers identified Jenkins `2.426.3`
- Authenticated live overview request succeeded and returned 38 visible jobs from the `/ci/ufrkpnsb/` folder
- Authenticated live job request succeeded for `ufr-kpnsb-product-request-workflow-service`
- Authenticated live build request succeeded for build `#1957` with result `SUCCESS`
- The live job payload includes useful build description metadata such as `branch`, `version`, and `serviceName`
- After redeploy, `tools/list` on `http://127.0.0.1:1985/mcp` exposes `get_jenkins_status`, `get_jenkins_overview`, `get_jenkins_job`, and `get_jenkins_build`
- After container recreate with `SslVerify=false`, `get_jenkins_job` succeeds through the live `utk-mcp` MCP endpoint
- After redeploy, `get_jenkins_build` returns a `changes` array; build `#1951` includes one git commit with commit id, message, author, email, and affected paths
- After redeploy, `get_bitbucket_branch` returns branch data and build-status entries; `feature/UTK2-3243-dev` resolves to latest commit `8cc4021bd9a8f0f46c4de52eb10a3931ba4970ba` with one `SUCCESSFUL` Jenkins build `#1952`
- The live `get_bitbucket_branch` response now includes `buildId: "1952"` for that Jenkins build entry
- After the config split and rebuild, the live MCP server still resolves Jenkins status to `https://platform/ci/ufrkpnsb/`
- After the config split and rebuild, a live `get_jenkins_job` call still succeeds for `ufr-kpnsb-product-request-workflow-service`
- After redeploy, `get_jenkins_build_log` returns `consoleTextUrl`, `lineCount`, and the full log; build `#1952` returned `4711` lines and included `Finished: SUCCESS`
- After redeploy, `get_jenkins_job_builds` returns filtered build lists; for `ufr-kpnsb-product-request-workflow-service` on `2026-03-31` with `limit=15`, it returned 10 builds from `#1957` through `#1948`
- After redeploy, `get_jenkins_job_builds(limit=10)` returns `changesSummary` in the live payload; recent builds now expose commit-message summaries without requiring per-build follow-up calls
- After redeploy, `get_jira_issue(issueKey, customFields=[\"Ссылки на ПР-ы\"])` resolves the display name to `customfield_38871` and returns the PR link in `customFields[0].value`
- After redeploy, `get_bitbucket_compare_diff` works for the incoming `ufr-lm-restructuring-service` compare URL and returned `diffCount: 117`

## Follow Up

- Decide whether the next Jenkins slice should expose queue state or richer pipeline metadata
