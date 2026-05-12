---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-28
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# 2026-04-28 helsinki-nginx-filter-frankfurt

## Goal

- Exclude failed requests from `45.134.217.102` (`frankfurt.markul.net`) from the `helsinki.markul.net` failed-request nginx log

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Local repo path: `/home/marat/dev/git/markul/docker`
- Local proxy config: `/home/marat/dev/git/markul/docker/docker-compose/helsinki.markul.net/nginx-proxy/config/custom-settings.conf`
- Live proxy path: `/home/marat/dev/git/docker/docker-compose/nginx-proxy`

## Actions

- Read the existing Helsinki failed-request logging session notes to reuse the current nginx and Loki layout
- Located the active filter in `custom-settings.conf`, where `map $status $log_failed` controlled writes to `/var/log/nginx/custom/failed-requests.log`
- Replaced the single status-only map with:
  - `map $status $log_failed_status`
  - `map $remote_addr $log_failed_source`
  - `map "$log_failed_status:$log_failed_source" $log_failed`
- Configured `45.134.217.102` to return `0` from the source map so that IP is excluded from failed-request logging regardless of response status, while other sources still follow the existing `2xx/3xx` suppression rule
- Synced the updated `custom-settings.conf` to `/home/marat/dev/git/docker/docker-compose/nginx-proxy/config/custom-settings.conf` on `helsinki`
- Confirmed `nginx -t` passed inside the running `nginx-proxy` container
- Reloaded nginx in-place with `nginx -s reload`
- Used the recurring `hub-arm.markul.net` `401` requests from `Watchtower (Docker)` as the live verification source after reload

## Decisions

- Keep the filter exception only on `helsinki` because the request was specific to the Frankfurt-to-Helsinki traffic path
- Exclude the known trusted VPS by source IP at nginx write time instead of filtering later in Alloy or Loki, so the noise never reaches the failed-request file or logging pipeline

## Validation

- `ssh helsinki.markul.net 'docker exec nginx-proxy nginx -t'`
- `ssh helsinki.markul.net 'docker exec nginx-proxy nginx -s reload'`
- `ssh helsinki.markul.net 'wc -l /home/marat/dev/git/docker/docker-compose/nginx-proxy/logs/nginx-proxy/failed-requests.log; sleep 35; wc -l /home/marat/dev/git/docker/docker-compose/nginx-proxy/logs/nginx-proxy/failed-requests.log; tail -n 8 /home/marat/dev/git/docker/docker-compose/nginx-proxy/logs/nginx-proxy/failed-requests.log'`

## Outcome

- After reload, recurring `hub-arm.markul.net` `401` requests from `45.134.217.102` no longer increased `failed-requests.log`
- Unrelated failed traffic from other IPs still appended normally, confirming the change removed only the intended self-generated Frankfurt noise
