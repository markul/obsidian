---
tags:
  - agents/project
note-type: agent-project
status: active
created: 2026-03-31T00:00
completed: 
related-ticket: 
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
  - [[work/alfa-bank/tickets/utk2-3243|UTK2-3243]]
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
- Kibana support now includes `get_kibana_status`, `list_kibana_indices`, and `search_kibana_logs`
- Large Bitbucket compare/PR diffs and Jenkins logs now export full payloads to local files and return summary metadata plus `SavedPath`
- Kibana log search currently goes through `api/console/proxy` with a compact normalized hit model and accepts a Kibana root or space-root base URL; `app/home` URLs are normalized down to the usable space root
- Kibana search no longer depends on a single default index pattern; it resolves `indexPattern` from `Kibana:IndexPatterns:{space}:{environment}` and still allows an explicit override
- Live local validation on `2026-04-18`: the updated server on `127.0.0.1:1986` reached `https://kibana-test/s/skp/` successfully, resolved `prelive -> app-k8s-prelive-skp-*`, and executed the MCP search with the new `environment`-based contract
- Live validation on `2026-04-18`: the Docker-backed MCP on `127.0.0.1:1985` now matches the local `127.0.0.1:1986` Kibana behavior after fixing the local `.env` Kibana username escaping; the container had been sending a doubled backslash in the domain username and getting `401 Unauthorized` from Kibana
- Live discovery on `2026-04-18`: `list_kibana_indices` against `environment="prelive"` showed the configured `app-k8s-prelive-skp-*` pattern has no matching docs, while an override `indexPattern="*"` exposed active prelive families including `app-ufr-k8s-app-prelive-*`
- Kibana search now also includes `app_id` in the searchable field set, uses it as a fallback for the normalized `Service` field when `service.name` is absent, and supports dedicated exact `service` plus `severity` filters
- Kibana service/app matching now uses `app_id` and `app` only for filtering; `service.name` is no longer used as a filter input and remains only as a fallback display field when app fields are absent
- Kibana hit normalization now uses only `app_id` and `app` for the normalized `Service` value; `service.name` is no longer used as a display fallback
- Live validation on `2026-04-18` against override `indexPattern="elk-test-cluster2:logs-skp-k8s-prelive-skp"` returned exact `Error` hits for `service="ufr-kpnsb-limit-service"` and surfaced a repeated PostgreSQL insert failure caused by missing column `sb_report_date`
- Repo docs were synced with the implemented tool surface; `README.md` is currently Russian by request and `AGENTS.md` holds the technical-gap list for contributors and agents
- JSON handling now forces UTF-8 decoding for JSON responses and uses readable non-escaped UTF-8 JSON serialization for MCP responses and exported JSON artifacts
- Local Docker compose runs the container as the host user to avoid root-owned files in `./downloads`
- The local codebase version is now `0.4.0`; unit validation passes with `29` tests
- [[agents/sessions/2026-04-19-utk-mcp-kibana-multi-service|2026-04-19]]: `SearchKibanaLogs` now accepts a `services` list (multi-service log trace); see [[agents/sessions/2026-04-19-kibana-covenant-errors|2026-04-19]] for covenant-monitoring-service error analysis

## Key Decisions

- Treat the configured Jenkins base URL as a readable folder resource, not only a job-path prefix
- Prefer Jenkins Git `BuildData.lastBuiltRevision` over `changeSets` when deriving top-level SCM metadata
- Keep user-facing runtime and setup documentation in `README.md`, and keep contributor/agent-only technical gaps in `AGENTS.md`
- Keep `get_bitbucket_pull_request_changes` lightweight as a changed-file list and expose actual PR diff content through a separate file-backed `get_bitbucket_pull_request_diff` tool
- Use explicit readable UTF-8 JSON serialization for MCP responses and exported JSON artifacts where non-ASCII text may appear
- For Kibana, treat the configured base URL as the Kibana root or space root and normalize pasted `.../app/home#...` URLs down to that root before calling API endpoints
- Resolve Kibana index patterns from a committed `{space -> environment -> indexPattern}` map in `appsettings*.json`; keep `indexPattern` as an optional MCP override instead of the default path
- Keep Kibana index discovery on a `_search` aggregation over `_index`; the current Kibana Console proxy in this environment returns `404` for `_cat` and `_resolve` endpoints
- Use dedicated term filters for Kibana `service` and `severity` constraints instead of relying on free-text queries when the goal is exact log scoping
- In the local Docker `env_file`, keep the Kibana domain username with a single backslash; doubling it is treated as a literal extra slash and breaks Kibana Basic auth
- Keep Kibana service normalization aligned with exact service filtering: only `app_id` and `app` define the normalized service name

## Blockers

- None currently
