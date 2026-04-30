---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-18
service: "[[work/alfa-bank/services/utk-mcp|utk-mcp]]"
related-project: "[[agents/projects/utk-mcp|UTK MCP]]"
related-ticket: 
---

# 2026-04-18 UTK MCP Kibana provider

## Goal

- Add an initial Kibana provider slice to `utk-mcp` so the server can report Kibana config state and search logs

## Scope

- Agent project: [[agents/projects/utk-mcp|UTK MCP]]
- Repo: `/home/marat/dev/git/alfa/utk-mcp`
- Related notes:
  - [[daily/2026-04-18]]

## Actions

- Reviewed the existing provider pattern across Jira, Confluence, Bitbucket, and Jenkins
- Added `Kibana` to the provider enum, provider status composition, DI registration, API tool registration, and env/appsettings config
- Added `IKibanaClient`, `KibanaTools`, and Kibana result models for compact normalized log hits
- Implemented `KibanaClient` with:
  - `get_kibana_status`
  - `search_kibana_logs`
  - base-url normalization from pasted `.../app/home#...` URLs to a usable Kibana root or space root
  - log search through `api/console/proxy` using Basic auth and `kbn-xsrf`
- Normalized search hits into a stable MCP response shape with timestamp, level, message, service, environment, trace id, request id, host, and logger
- Added unit tests for:
  - provider status aggregation
  - provider enum expectations
  - Kibana search request/response mapping
  - Kibana default index-pattern fallback
- Fixed one JSON-body construction compile error and one `HttpListener` cleanup issue during test execution
- Ran the full unit test suite to green
- Updated local `.env` with Kibana space-root config for `https://kibana-test/s/skp/`
- Started the updated local server on `http://127.0.0.1:1986` because `1985` was already occupied by an older `0.2.0` instance
- Validated the MCP transport directly over HTTP:
  - `tools/list` exposed `get_kibana_status` and `search_kibana_logs`
  - `get_kibana_status` reported the expected configured base URL
  - `search_kibana_logs` with `indexPattern="*"` and `limit=3` returned live hits from `app-ufr-app-prelive-001333`
- Reworked Kibana search configuration from a single default index pattern to `Kibana:IndexPatterns:{space}:{environment}`
- Changed the MCP contract so `search_kibana_logs` requires `environment` and only accepts `indexPattern` as an optional override
- Added committed `skp` mappings for:
  - `dev -> app-k8s-dev-skp-*`
  - `int -> app-k8s-int-skp-*`
  - `prelive -> app-k8s-prelive-skp-*`
- Re-ran the full unit suite after the contract change
- Revalidated the live MCP call on `127.0.0.1:1986` with `environment="prelive"` and no override; the server resolved `space="skp"` and `indexPattern="app-k8s-prelive-skp-*"` correctly
- Added `list_kibana_indices` to stop guessing concrete prelive targets
- First implementation attempt used `_cat/indices`, then `_resolve/index`; both returned `404` through this Kibana Console proxy
- Switched the discovery tool to a `_search` aggregation on `_index`, which works in this environment because `_search` already works through the proxy
- Re-ran the unit suite after the discovery-tool changes
- Live discovery results:
  - `environment="prelive"` with the configured map returned `0` concrete indices for `app-k8s-prelive-skp-*`
  - `environment="prelive"` with override `indexPattern="*"` returned `32` concrete indices
  - active prelive families in the wildcard result include `app-ufr-k8s-app-prelive-*` and `app-ufr-app-prelive-*`
  - searching `app-ufr-k8s-app-prelive-*` still did not find the expected `ufr-kpnsb-limit-service` name fragments, so the next blocker is service-name/field discovery rather than raw index discovery
- Updated the search field set so free-text service queries also hit `app_id`
- Revalidated `app-ufr-k8s-app-prelive-*` with `ufr-kpnsb-limit-service` and with an error-focused query; both still returned `0` hits, so the likely remaining issue is that the real `app_id` value differs from the guessed service name
- Added an optional `severity` filter to `search_kibana_logs` and implemented it as a term-based filter across `severity`, `severity_text`, `log.level`, and `level` variants
- Added an optional exact `service` filter to `search_kibana_logs` and implemented it as a term-based filter across `app_id` and `app` variants so service scoping no longer depends on free-text message matches
- Tightened Kibana service/app matching so both the exact `service` filter and the free-text service/app field list use only `app_id` and `app`; `service.name` is now only a normalized-output fallback
- Re-ran the full unit suite after the `service` and `severity` filter changes
- Revalidated live search against the user-provided Kibana view pattern override `elk-test-cluster2:logs-skp-k8s-prelive-skp`
- Live `search_kibana_logs` with `environment="prelive"`, `service="ufr-kpnsb-limit-service"`, `severity="Error"`, and a three-day window returned the last `10` exact error hits for the target service
- The latest returned error chain shows an EF Core `DbUpdateException` caused by PostgreSQL `42703`: column `sb_report_date` does not exist in relation `client_limit`
- Debugged why the Codex-facing `utk-mcp` wrapper still failed for Kibana while the local `127.0.0.1:1986` server worked
- Reproduced the split directly over MCP HTTP:
  - `127.0.0.1:1986` returned live `dev` hits for `ufr-kpnsb-ews-check-service`
  - `127.0.0.1:1985` returned generic MCP errors for the same calls
- Checked the Docker-backed `1985` container logs and confirmed the real failure was Kibana `401 Unauthorized`
- Compared runtime env values and found the cause:
  - local `1986` used `MOSCOW\U_M29R2`
  - Docker `1985` used `MOSCOW\\U_M29R2`
- Fixed the local repo `.env` so the Kibana username uses a single backslash, recreated the Docker container, and revalidated Kibana on `127.0.0.1:1985`
- Re-ran the original user query through the Codex `utk-mcp` wrapper after the recreate; it returned the last `50` `dev` logs for `ufr-kpnsb-ews-check-service`
- Removed `service.name` as a fallback for the normalized Kibana `Service` field so output semantics now match the exact filter semantics based on `app_id` and `app` only
- Added a unit test that proves a hit with only `service.name` no longer produces a normalized `Service` value

## Decisions

- Start with a minimal read-only Kibana slice instead of attempting saved-search, export, or context-thread features in the first pass
- Use the Kibana Console proxy for the initial search implementation so the provider can work from a Kibana base URL without introducing a separate Elasticsearch endpoint yet
- Accept the Kibana root or space root as configuration and normalize pasted UI URLs down to the API-usable root
- Use exact term filters for Kibana service scoping when the operator wants logs for one service, because free-text search also matches upstream/downstream call messages from other services
- Treat Windows-domain usernames in Docker `env_file` as literal strings; do not double-escape the backslash for Kibana credentials
- Keep Kibana output normalization strict: `service.name` should not backfill the normalized service field when exact filtering is defined on `app_id` and `app`

## Validation

- `dotnet test /home/marat/dev/git/alfa/utk-mcp/UtkMcp.sln`
- Result: `26` passed
- `curl http://127.0.0.1:1986/health`
- Result: `{"status":"ok","name":"utk-mcp","version":"0.3.0"}`
- MCP `tools/call` → `get_kibana_status`
- Result: configured `Kibana` provider for `https://kibana-test/s/skp/`
- MCP `tools/call` → `search_kibana_logs` with `indexPattern="*"` and `limit=3`
- Result: `3` live hits returned; `total=10000`, `totalRelation="gte"`
- MCP `tools/call` → `search_kibana_logs` with `environment="prelive"` and no override
- Result: resolved `space="skp"` and `indexPattern="app-k8s-prelive-skp-*"` successfully; returned `0` hits for the sampled query window
- `dotnet test /home/marat/dev/git/alfa/utk-mcp/UtkMcp.sln`
- Result: `27` passed
- MCP `tools/call` → `list_kibana_indices` with `environment="prelive"`
- Result: configured pattern `app-k8s-prelive-skp-*` returned `0` concrete indices
- MCP `tools/call` → `list_kibana_indices` with `environment="prelive"` and override `indexPattern="*"`
- Result: `32` concrete indices discovered; wildcard results include `app-ufr-k8s-app-prelive-001181` through `001188` and `app-ufr-app-prelive-001326` through `001333`
- `dotnet test /home/marat/dev/git/alfa/utk-mcp/UtkMcp.sln`
- Result: `27` passed
- MCP `tools/call` → `search_kibana_logs` with `environment="prelive"`, `indexPattern="elk-test-cluster2:logs-skp-k8s-prelive-skp"`, `service="ufr-kpnsb-limit-service"`, `severity="Error"`, and `limit=10`
- Result: `10` exact hits returned for `ufr-kpnsb-limit-service`; latest entries include `DbUpdateException`, `Failed executing DbCommand`, and PostgreSQL `42703` for missing column `sb_report_date`
- `docker logs utk-mcp-utk-mcp-1`
- Result: Kibana failures on `127.0.0.1:1985` were real `401 Unauthorized` responses, with one earlier transient DNS failure during the broken state
- MCP `tools/call` on `http://127.0.0.1:1986/mcp` → `list_kibana_indices(environment="dev")`
- Result: returned live `dev` backing indices for `elk-test-cluster2:logs-skp-k8s-dev-skp`
- MCP `tools/call` on `http://127.0.0.1:1986/mcp` → `search_kibana_logs(environment="dev", service="ufr-kpnsb-ews-check-service", limit=3)`
- Result: returned live hits, proving the query itself was valid and the breakage was specific to the Docker-backed instance
- `docker compose up -d --force-recreate utk-mcp`
- Result: recreated the local Docker-backed MCP with the corrected Kibana username env value
- MCP `tools/call` on `http://127.0.0.1:1985/mcp` → `list_kibana_indices(environment="dev")`
- Result: returned live `dev` backing indices after the env fix
- MCP `tools/call` on `http://127.0.0.1:1985/mcp` → `search_kibana_logs(environment="dev", service="ufr-kpnsb-ews-check-service", limit=3)`
- Result: returned live hits after the env fix
- Codex `utk-mcp` tool → `search_kibana_logs(environment="dev", service="ufr-kpnsb-ews-check-service", limit=50)`
- Result: returned `50` hits through the same wrapper path that had previously failed
- `dotnet test /home/marat/dev/git/alfa/utk-mcp/UtkMcp.sln --filter KibanaClientTests`
- Result: `5` passed, including the new assertion that `service.name` alone does not populate the normalized `Service` field
