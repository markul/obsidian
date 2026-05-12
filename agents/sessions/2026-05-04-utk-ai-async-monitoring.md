---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-04
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-04 UTK AI Async Monitoring

## Goal

- Make manual monitoring triggers return quickly and let a background job fulfill queued runs, then notify connected Web clients when processing finishes

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[daily/2026-05-04]]

## Actions

- Changed trigger handling so `POST /api/monitoring-runs` creates `MonitoringRun` records with status `New` and returns `202 Accepted` with those records without doing MCP/Kibana or AlfaGen work inline.
- Added a hosted `MonitoringRunWorker` that polls `New` runs, marks each run `Pending` before work starts, then sets `Done` or `Error` after processing.
- Moved the previous Kibana search, LLM analysis, finding persistence, and Kibana URL construction into `MonitoringRunProcessor`.
- Added an in-process monitoring completion bus with two consumers: one SignalR consumer for connected Web clients and one console-log consumer.
- Added `/hubs/monitoring` SignalR endpoint and a Web SignalR client that listens for `MonitoringRunFinished`, shows a completion message, and refreshes the findings list.
- Fixed the first SignalR refresh follow-up: completion events now include processed services, and the Web client tracks queued run ids so it refreshes only for the completed run/current service scope and reports the loaded finding count.
- Hardened the Web SignalR callback after an incoming completion message did not refresh: the callback now returns the refresh task to SignalR, tolerates older payloads without `services`, and surfaces refresh exceptions in the status line.
- Fixed a follow-up UI hang where the Web page stayed on `Monitoring finished. Refreshing results...`: the SignalR handler now requests a final render after the awaited refresh and bounds the refresh with a 30-second timeout.
- Replaced inline active/loading/error/completion status text with bottom-right monitoring toast popups; a toast appears when a monitoring run is queued, shows spinner plus service/date/environment, and is removed when the matching SignalR completion arrives.
- Added intermediate SignalR progress notifications from `MonitoringRunProcessor`; active monitoring toasts now update through `MonitoringRunProgress` while the worker moves through startup, Kibana search, supplement search, sampling, AlfaGen analysis, and persistence.
- Follow-up on 2026-05-05: error completions now keep an error-state toast visible with the run error message, and immediate trigger API failures also create an error toast.
- Follow-up on 2026-05-05: added `MonitoringStrategy` with default raw search and optional aggregated search. Aggregated search calls `aggregate_kibana_logs`, flattens representative bucket examples for AlfaGen, persists runs under a separate strategy, and the Web UI switch filters/triggers the chosen strategy.
- Follow-up on 2026-05-05: changed aggregated `groupBy` to `app_id` after replaying the latest MCP aggregation response; `app_id.keyword` and `log.logger.keyword` were missing and collapsed all logs into one `<missing>` bucket.

## Decisions

- Keep the queue durable in PostgreSQL using `MonitoringRun.Status=New` rather than relying only on an in-memory channel.
- Keep legacy enum aliases for old status names while new writes use `New`, `Pending`, `Done`, and `Error`.
- Use an in-process event bus for the first version; cross-process delivery can be introduced later if the API scales out.
- Include service scope in monitoring completion payloads so the Web UI does not react to unrelated runs for the same date/environment.
- Keep raw search unchanged as the default strategy; use a separate persisted strategy value for aggregated sampling so comparison runs do not delete each other's findings.

## Validation

- `dotnet build Utk.Ai.slnx`

## Current State

- The solution builds after the async trigger and SignalR notification changes.
- Runtime smoke testing was not run in this pass because the developer wanted to restart Web manually.
