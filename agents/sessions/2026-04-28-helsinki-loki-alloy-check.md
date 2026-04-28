---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-28
related-project: "[[agents/projects/my-infra|My Infra]]"
related-ticket:
---

# 2026-04-28 helsinki-loki-alloy-check

## Goal

- Re-check the live `Loki + Alloy` failed-request logging setup on `helsinki.markul.net`

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Host: `helsinki.markul.net`
- Local repo path: `/home/marat/dev/git/markul/docker/docker-compose/helsinki.markul.net/logging`
- Live compose path: `/home/marat/dev/git/docker/docker-compose/helsinki-logging`
- Live nginx failed log: `/home/marat/dev/git/docker/docker-compose/nginx-proxy/logs/nginx-proxy/failed-requests.log`

## Actions

- Read the vault policy and resumed the prior logging context from [[agents/sessions/2026-04-23-helsinki-nginx-failed-loki]]
- Re-checked the durable infra state in [[agents/projects/my-infra|My Infra]]
- Compared the tracked local `helsinki.markul.net/logging` files with the live host files under `/home/marat/dev/git/docker/docker-compose/helsinki-logging`
- Confirmed over SSH that `loki` and `alloy-logs` are both running in the live compose project
- Verified that the live `alloy-logs` container still mounts:
  - `/home/marat/dev/git/docker/docker-compose/helsinki-logging/config.alloy` -> `/etc/alloy/config.alloy`
  - `/home/marat/dev/git/docker/docker-compose/helsinki-logging/geoip` -> `/etc/alloy/geoip`
  - `/home/marat/dev/git/docker/docker-compose/nginx-proxy/logs/nginx-proxy` -> `/var/log/nginx/custom`
- Checked recent `alloy-logs` output and confirmed the file source restarted cleanly on `2026-04-26` and resumed `start tailing file` for `/var/log/nginx/custom/failed-requests.log`
- Checked Alloy metrics and confirmed:
  - `loki_source_file_files_active_total=1`
  - `loki_write_sent_entries_total=9211`
  - `loki_write_batch_retries_total=0`
  - all `loki_write_dropped_entries_total` reasons at `0`
- Checked the live failed-request log and confirmed recent `hub-arm.markul.net` `401` entries from `Watchtower (Docker)`
- Queried Loki directly on the host and confirmed:
  - `/ready` returned `200 OK`
  - `/loki/api/v1/labels` returned the expected failed-request label set
  - recent `hub-arm.markul.net` `401` entries were queryable immediately with `job`, `host`, `method`, `status`, `source_host`, and GeoIP country labels
- Queried the 24-hour total and confirmed Loki currently stores `6328` failed-request entries over the last day
- Checked city/subdivision label values and found `geoip_city_name` and `geoip_subdivision_name` currently have zero retained values even though the Alloy config still declares those labels
- Verified from the current external environment that `http://helsinki.markul.net:3100/ready` and `/loki/api/v1/labels` are reachable
- Tried to re-read `ufw` on the host, but `sudo` required an interactive password, so the current firewall rule set was not re-verified

## Decisions

- Treat the live `Loki + Alloy` setup on `helsinki` as healthy without making config changes
- Treat country-level enrichment as verified and city/subdivision enrichment as currently unverified in retained data
- Keep the firewall exposure claim cautious until the host `ufw` rules are re-checked with elevated access

## Validation

- `ssh helsinki.markul.net 'cd /home/marat/dev/git/docker/docker-compose/helsinki-logging && docker compose ps'`
- `ssh helsinki.markul.net 'docker logs --tail 80 alloy-logs'`
- `ssh helsinki.markul.net 'curl -sS http://127.0.0.1:12345/metrics | grep -E "loki_write_(sent_entries_total|dropped_entries_total|batch_retries_total)|loki_source_file_files_active_total"'`
- `ssh helsinki.markul.net 'curl -sS -i http://127.0.0.1:3100/ready'`
- `ssh helsinki.markul.net 'curl -sS http://127.0.0.1:3100/loki/api/v1/labels'`
- `ssh helsinki.markul.net 'curl -sS -G --data-urlencode "query={job=\"nginx-failed-requests\",host=\"hub-arm.markul.net\",status=\"401\"}" --data-urlencode "limit=5" --data-urlencode "direction=backward" http://127.0.0.1:3100/loki/api/v1/query_range'`
- `curl -sS -i http://helsinki.markul.net:3100/ready`

## Outcome

- The documented `helsinki` failed-request path is still live end to end: `nginx-proxy` failed log -> Alloy file tail -> Loki ingestion -> queryable streams
- The current setup is healthy at the service level and is actively ingesting fresh `401` traffic from `hub-arm.markul.net`
- The only unresolved point from this check is whether city/subdivision GeoIP labels are still being populated in practice and whether host port `3100` is still restricted exactly as previously documented
