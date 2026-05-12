---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-09
service: "[[work/alfa-bank/services/utk-mcp|utk-mcp]]"
project: "[[agents/projects/utk-mcp|UTK MCP]]"
ticket:
---

# 2026-05-09 UTK MCP Document Links

## Goal

- Let agents and operators open concrete Kibana log entries returned by MCP tools directly in Discover.

## Scope

- Agent project: [[agents/projects/utk-mcp|UTK MCP]]
- Repo: `/home/marat/dev/git/alfa/utk-mcp`

## Actions

- Added optional `documentId` to `search_kibana_logs` and mapped it to an Elasticsearch `ids` filter.
- Added `documentId` to `KibanaSearchLogsResult` so callers can see the applied lookup key.
- Added `kibanaUrl` to `KibanaLogEntryResult`; raw search hits and aggregation representative examples share the same mapper.
- Added `Kibana:DiscoverDataViewIds` config and configured `skp/dev` as `7cbdb608-c31f-5d13-8341-d5cb8d39518d`.
- Built Discover URLs with a ±10 minute window around the log timestamp, `app_id` and `severity` phrase filters, and a Lucene `_id:"..."` query.
- Updated `README.md` and `AGENTS.md` for the new `documentId` and `kibanaUrl` behavior.
- Investigated invalid `kibanaUrl` values surfaced through `utk-ai` chat history: the local MCP was returning the FQDN request host and raw `_id:"..."` query, while a known-good Kibana URL used the short `https://kibana-test/s/skp/` host and `_id:%22...%22`.
- Added `Kibana:DiscoverBaseUrl` so Dockerized MCP can keep the FQDN `BaseUrl` for server-side Kibana requests while returning short browser-facing Discover links.
- Changed generated Discover `_id` queries to URL-encode the surrounding quotes as `%22`, matching Kibana-generated URLs.

## Validation

- `dotnet build UtkMcp.sln --no-restore --disable-build-servers` passed.
- `dotnet test UtkMcp.sln --no-build --disable-build-servers` passed.
- `git diff --check` passed.
- Rebuilt and restarted local Docker MCP with `UID=$(id -u) GID=$(id -g) docker compose up -d --build`.
- `curl -fsS http://127.0.0.1:1985/health` passed.
- Live MCP `search_kibana_logs` with `documentId=7K8NCp4B3qZkRnSBWFt2` returned one hit with `kibanaUrl`.
- Live MCP `aggregate_kibana_logs` with one bucket and one example returned the representative example with `kibanaUrl`.
- `dotnet test UtkMcp.sln --no-restore --disable-build-servers` passed after the URL fix (`34` tests).
- `git diff --check` passed after the URL fix.
- Rebuilt and restarted local Docker MCP with `docker compose up -d --build`.
- Live MCP `search_kibana_logs` for `documentId=yX5fCZ4B3qZkRnSBjBtA`, `ufr-kpnsb-product-request-workflow-service`, `Error`, and `2026-05-08T20:45:16.062Z..2026-05-08T21:05:16.062Z` returned one hit whose `kibanaUrl` starts with `https://kibana-test/s/skp/app/discover#/?_g=` and contains `query:'_id:%22yX5fCZ4B3qZkRnSBjBtA%22'`.

## Decisions

- Keep document-id lookup separate from free-text `query` so exact document retrieval is deterministic.
- Generate Discover URLs inside MCP, not downstream clients, because MCP already knows the Kibana base URL, space, environment, and data view id.
- Keep request `BaseUrl` and browser-facing `DiscoverBaseUrl` separate for local Docker/VPN setups where DNS requirements differ.

## Follow-Up

- None currently.
