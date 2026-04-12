---
tags:
  - agents/project
  - status/active
---

# UTK MCP

## Status

- `active`

## Timestamps

- Created: `2026-03-31 00:00`
- Completed: `not completed`

## Goal

- Extend and validate the `utk-mcp` server for internal Alfa systems

## Plan

1. Keep the running local MCP instance aligned with repo changes after meaningful capability and serialization updates.
2. Choose the next capability slice based on the current technical gaps and the highest-value provider workflow.
3. Continue validating new tools against the live local MCP server after rebuilds.

## Scope

- Repository: `utk-mcp`
- Bitbucket repository: [utk-mcp](https://git.moscow.alfaintra.net/users/u_m29r2/repos/utk-mcp/)
- Local path: `/home/marat/dev/git/alfa/utk-mcp`
- Related work:
  - [[work/alfa-bank/tickets/on-hold/utk2-3243|UTK2-3243]]
  - [[agents/projects/utk2-3243|UTK2-3243 Agent Work]]

## Validation

- `docker compose up -d --build`
- `curl http://127.0.0.1:1985/health`
- `curl http://127.0.0.1:1985/`
- `dotnet test UtkMcp.sln`

## Current State

- Purpose: MCP server for internal Alfa systems including Jira, Confluence, Bitbucket, and Jenkins
- Jira support includes `get_jira_issue` with optional custom fields and `get_jira_development_metadata`
- Confluence support includes `get_confluence_page`, `search_confluence_content`, `list_confluence_page_attachments`, and `download_confluence_page_attachment`
- Jenkins support includes `get_jenkins_overview`, `get_jenkins_job`, `get_jenkins_job_builds`, `get_jenkins_build`, and `get_jenkins_build_log`
- Jenkins build responses expose top-level `CommitHash`, normalized `BranchName`, and `BranchUrl`
- Bitbucket support includes `get_bitbucket_branch`, `get_bitbucket_compare_diff`, and `get_bitbucket_pull_request_diff`
- Large Bitbucket compare/PR diffs and Jenkins logs now export full payloads to local files and return summary metadata plus `SavedPath`
- Repo docs were synced with the implemented tool surface; `README.md` is currently Russian by request and `AGENTS.md` holds the technical-gap list for contributors and agents
- JSON handling now forces UTF-8 decoding for JSON responses and uses readable non-escaped UTF-8 JSON serialization for MCP responses and exported JSON artifacts
- Local Docker compose runs the container as the host user to avoid root-owned files in `./downloads`
- The running local MCP instance is on version `0.2.0` and the root endpoint reports `streamable-http`

## Key Decisions

- Treat the configured Jenkins base URL as a readable folder resource, not only a job-path prefix
- Prefer Jenkins Git `BuildData.lastBuiltRevision` over `changeSets` when deriving top-level SCM metadata
- Keep user-facing runtime and setup documentation in `README.md`, and keep contributor/agent-only technical gaps in `AGENTS.md`
- Keep `get_bitbucket_pull_request_changes` lightweight as a changed-file list and expose actual PR diff content through a separate file-backed `get_bitbucket_pull_request_diff` tool
- Use explicit readable UTF-8 JSON serialization for MCP responses and exported JSON artifacts where non-ASCII text may appear

## Blockers

- None currently
