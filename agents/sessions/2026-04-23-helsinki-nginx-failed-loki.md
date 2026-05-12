---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-23
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# Helsinki Nginx Failed Requests Loki

## Goal

Continue `docker-compose/helsinki.markul.net/logging/plan.md` after steps 1-4 and complete steps 5-9: switch Alloy from `nginx-proxy` Docker JSON logs to the dedicated failed-request file, parse useful labels, add GeoIP, deploy to `helsinki`, and validate end to end.

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Local repo path: `/home/marat/dev/git/markul/docker`
- Local logging config: `/home/marat/dev/git/markul/docker/docker-compose/helsinki.markul.net/logging`
- Live logging path: `/home/marat/dev/git/docker/docker-compose/helsinki-logging`
- Live proxy path: `/home/marat/dev/git/docker/docker-compose/nginx-proxy`

## Actions

- Read the existing plan and prior session context from [[agents/sessions/2026-04-21-my-infra-proxmox-node-exporter-readiness]].
- Confirmed the live `alloy-logs` container was still using the old Docker JSON pipeline even though the local repo had an updated file-tail config.
- Patched `config.alloy` because Alloy `v1.15.0` rejected the draft `stage.geoip` shape; the working syntax uses `db = "/etc/alloy/geoip/ip66.mmdb"` and `db_type = "city"`.
- Made the first access-log regex parse the whole quoted request into `request`, `status`, and surrounding fields, then parse `method` and `path` from `request` separately so malformed `400` requests can still keep status/host labels.
- Validated the candidate config with `grafana/alloy:v1.15.0 validate`.
- Synced the local logging directory to `/home/marat/dev/git/docker/docker-compose/helsinki-logging` on `helsinki`.
- Recreated only `alloy-logs` with `docker compose up -d alloy`; `loki` stayed running.
- Added local dashboard definition at `/home/marat/dev/git/markul/docker/docker-compose/dev.markul.net/monitoring/grafana/dashboards/failed-requests.json`.
- Imported the dashboard into Dockerized Grafana on `dev.markul.net` via the Grafana API as `Failed Requests`, UID `failed-requests`.

## Validation

- `alloy-logs` started cleanly and logged `start tailing file` for `/var/log/nginx/custom/failed-requests.log`.
- Generated `https://ftp.markul.net/codex-alloy-failed-1776936188` from `dev.markul.net`; it returned `502`.
- The host `failed-requests.log` line count increased by exactly one.
- The matching line was:

```text
45.134.217.102 - ftp.markul.net [23/Apr/2026:09:23:10 +0000] "GET /codex-alloy-failed-1776936188 HTTP/2.0" 502 157 "-" "curl/8.5.0"
```

- Loki returned the new stream from both `helsinki` locally and `dev.markul.net` remotely with labels `job="nginx-failed-requests"`, `host="ftp.markul.net"`, `method="GET"`, `status="502"`, `geoip_country_code="DE"`, and `geoip_country_name="Germany"`.
- Alloy metrics after the test showed `loki_write_sent_entries_total` at `1`, `loki_write_batch_retries_total` at `0`, and all `loki_write_dropped_entries_total` reasons at `0`.
- Grafana datasource proxy query for `sum by (geoip_country_code, geoip_country_name) (count_over_time({job="nginx-failed-requests", source_host="helsinki.markul.net", geoip_country_code!=""}[24h]))` returned country groups including `DE`, `NL`, and `US`.
- Grafana API confirmed the shared dashboard exists at `/d/failed-requests/failed-requests`.
- After visual review, the country map/table panels were corrected from `labelsToFields` to `seriesToRows` transformations because `labelsToFields` duplicated country rows in Grafana 11.3.1.
- After another visual review, the aggregate country panels were changed from `queryType="range"` with `instant=true` to real `queryType="instant"` queries, because Grafana still returned multiple range samples per country and duplicated rows/markers.
- Final country-panel adjustment: instant Loki metric results already expose `geoip_country_code`, `geoip_country_name`, and `Value #A` fields, so the map now uses those native fields directly and the table only applies an `organize` rename/sort transform.
- Map marker color now uses failed-request count (`Value #A`) thresholds instead of country-code mappings, so new countries get automatic volume-based colors: green for `0-10`, yellow for `11-50`, orange for `51-100`, red for `101-500`, and dark red for `501+`.
- The Helsinki and dev dashboards were consolidated into one shared `Failed Requests` dashboard with a `Loki source` datasource variable. The old `helsinki-nginx-failed` and `dev-nginx-failed` dashboards were deleted. The selectable Loki datasources are `helsinki.markul.net` and `dev.markul.net Loki`; the dev datasource keeps `Loki` in its display name because Grafana already has a Prometheus datasource named `dev.markul.net`.
- Added city-level GeoIP enrichment to both Alloy pipelines by promoting `geoip_city_name`, `geoip_subdivision_code`, and `geoip_subdivision_name` after `stage.geoip`.
- Updated the shared Grafana dashboard to version 3 with a compact `Failed Requests By City` table using `sum by (geoip_country_name, geoip_subdivision_name, geoip_city_name)`.
- Validation: both local configs passed `grafana/alloy:v1.15.0 validate`, both live `alloy-logs` containers restarted cleanly and resumed tailing, and the dashboard import succeeded at `/d/failed-requests/failed-requests`. A fresh Helsinki test request was ingested after restart, but that source IP returned only country data from the MMDB, so city labels will appear only for IPs with city data in the database.

## Decisions

- Keep only low-cardinality parsed fields as Loki labels: `host`, `method`, `status`, and country-level GeoIP fields.
- Keep `client_ip`, `path`, `referer`, and `user_agent` out of Loki labels to avoid high cardinality.
- Leave Loki running during the Alloy config deploy because `loki-config.yml` did not change.

## Handoff

- Steps 1-10 in `docker-compose/helsinki.markul.net/logging/plan.md` are complete.
- The dashboard URL is `https://grafana.markul.net/d/failed-requests/failed-requests`.
- Older retained streams under `job="nginx-proxy-probes"` still exist until retention expires, and older `nginx-failed-requests` streams from before the city-label deploy do not have city labels.

## Dev Nginx Follow-Up

- Extended the same pattern to `dev.markul.net`, but with its own local Loki instead of writing into Helsinki Loki.
- Added `docker-compose/dev.markul.net/nginx-proxy/templates/nginx.tmpl`, mounted it into `nginx-proxy`, and added `/var/log/nginx/custom/failed-requests.log failed_only if=$log_failed`.
- Added `docker-compose/dev.markul.net/logging` with `loki-dev` (`grafana/loki:3.7.0`) and `alloy-logs` (`grafana/alloy:v1.15.0`) on the shared `nginx-proxy` network.
- Dev Loki is bound on the host as `127.0.0.1:3101` and reachable from Grafana as `http://loki-dev:3100`.
- Added Grafana datasource `dev.markul.net Loki` with UID `loki_dev`.
- Added local dev failed-request panels through the shared `Failed Requests` dashboard by selecting `dev.markul.net Loki` in the `Loki source` variable.
- Dev Alloy drops private client IPs before GeoIP enrichment using `private_client_ip` drop reason, because internal `192.168.*` Watchtower requests were otherwise mapped to bogus countries by the GeoIP database.
- Validation: `nginx -t` passed after recreating only `nginx-proxy`; `loki-dev` and `alloy-logs` are ready; Grafana datasource health for `loki-dev` is OK; a private local test request was written to the nginx file but did not appear in Loki after the private-IP drop filter.
