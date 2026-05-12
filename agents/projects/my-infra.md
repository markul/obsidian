---
tags:
  - agents/project
note-type: agent-project
status: active
created: 2026-04-02T19:22
completed: 
ticket: 
---

# My Infra

## Status

- `active`

## Timestamps

- Created: `2026-04-02 19:22`
- Completed: `not completed`

## Goal

- Track the home and VPS infrastructure under `markul.net`

## Plan

1. Watch the shared `Failed Requests` Grafana dashboard for a few days and tune panels if the country map, table shape, or Loki source selector needs adjustment.
2. Expand the same `Alloy` parsing path to other public ingress hosts after the shared dashboard and queries are stable.
3. Decide whether to keep or retire the older retained `job="nginx-proxy-probes"` Loki stream once its seven-day retention window expires.
4. Reconcile [[personal/tech/electricity-consumption|electricity-consumption]] with the current tariff and saved measurements.
5. Continue tightening the public SSH exposure and trust state for the externally reachable hosts.

## Scope

- Hub note: [[personal/index|Personal]]
- Area note: [[personal/tech/infrastructure|Infrastructure]]
- Related notes:
  - [[personal/tech/hardware|hardware]]
  - [[personal/tech/electricity-consumption|electricity-consumption]]
  - [[daily/2026-04-02]]
  - [[daily/2026-04-03]]
  - [[agents/sessions/2026-04-02-my-infra-bootstrap]]
  - [[agents/sessions/2026-04-02-my-infra-inventory-cleanup]]
  - [[agents/sessions/2026-04-02-my-infra-public-security-review]]
  - [[agents/sessions/2026-04-21-my-infra-proxmox-node-exporter-readiness]]
  - [[agents/sessions/2026-04-23-helsinki-nginx-failed-loki]]
  - [[agents/sessions/2026-04-24-infra-mtproto-proxy]]
  - [[agents/sessions/2026-04-24-optiplex-seerr-infra-dns]]
  - [[agents/sessions/2026-04-25-markul-docker-review]]
  - [[agents/sessions/2026-04-25-markul-local-systemd-resolved]]
  - [[agents/sessions/2026-04-25-media-stack-map]]
  - [[agents/sessions/2026-04-25-optiplex-seerr-radarr-base-url]]
  - [[agents/sessions/2026-04-26-helsinki-hub-arm-registry]]
  - [[agents/sessions/2026-04-26-markul-arm-registry-split]]
  - [[agents/sessions/2026-04-28-helsinki-loki-alloy-check]]
  - [[agents/sessions/2026-04-28-helsinki-nginx-filter-frankfurt]]

## Validation

## Current State

- This project is note-based inside the Obsidian vault; there is no separate `my-infra` repo tracked here
- [[personal/tech/infrastructure|Infrastructure]] is organized into `Overview`, `Network`, `Service Placement`, and `Machine Inventory`
- Current confirmed machines include `proxmox.markul.net`, `raspberrypi.markul.net`, `dev.markul.net`, `alfa.markul.net`, `infra.markul.net`, `optiplex.markul.net`, `frankfurt.markul.net`, and `helsinki.markul.net`
- `infra.markul.net` was re-checked live on `2026-04-24`: active compose files live under `/home/marat/docker/`, not under the tracked repo checkout path, and the host now has a dedicated `mtproto-proxy` compose project exposing `8443/tcp -> 443/tcp`
- The media/request split is now mapped in [[personal/tech/media-stack|Media Stack]]: `infra.markul.net` runs `Prowlarr + Radarr + Sonarr + qBittorrent`, while `optiplex.markul.net` runs `Jellyfin + Jellyseerr` against shared `uid=1000` media paths; the same pass also found an unsafe live `qBittorrent` `OnTorrentAdded` remote shell hook and two world-writable media roots on `infra.markul.net`
- `optiplex.markul.net` was re-checked live on `2026-04-25`: `seerr` still uses public DNS `9.9.9.9`, but static `extra_hosts` entries now cover `infra.markul.net`, `radarr.markul.net`, and `sonarr.markul.net`; the live Radarr integration failure was instead caused by a stale Jellyseerr `baseUrl` of `/radarr.markul.net` while Radarr currently serves at `/`
- `optiplex`, `dev.markul.net`, and `infra.markul.net` were re-checked live on `2026-04-25`: all three use `systemd-resolved` with router DNS `192.168.222.100`; ordinary unicast DNS names such as `qbit.markul.net` resolve, but `.local` names such as `qbit.markul.local` fail locally even though the router DNS answers them directly, because `systemd-resolved` treats `.local` as special local-link naming unless the domain is explicitly routed to unicast DNS
- The latest `markul/docker` review on `2026-04-25` found three concrete risks in commit `b941266`: the new Optiplex Kafka stack advertises `kafka:9092` and will not work for host or LAN clients, the new registry cleanup script only handles a narrow repository path shape and can miss common registry layouts, and the new `seerr` setup hardcodes public DNS plus static host mappings so future internal names still fail to resolve inside the container
- `raspberrypi.markul.net` is now inventoried from live SSH data
- `proxmox` at `192.168.222.3` was re-checked live on `2026-04-21`: Proxmox VE `8.4.5` on Debian `12` (`x86_64`) with `node_exporter 1.11.1` installed as a systemd service, listening on port `9100`, outbound GitHub access working, and `pve-firewall` currently `disabled/running`
- Grafana on `dev.markul.net` already has working `Prometheus` and `Elasticsearch` datasources, but the current Elasticsearch `logstash-*` data does not include `nginx-proxy` access logs or GeoIP-enriched probe events
- `helsinki.markul.net` currently has the clearest observed vulnerability-probe traffic in raw `nginx-proxy` logs, including requests for `/.env`, `/.git/*`, `xmlrpc.php`, `wlwmanifest.xml`, `/HNAP1`, `/solr/admin/*`, `/boaform/*`, and `/cgi-bin/authLogin.cgi`
- `helsinki.markul.net` now also runs a dedicated local logging stack under `/home/marat/dev/git/docker/docker-compose/helsinki-logging` with `grafana/loki:3.7.0` on host port `3100` and `grafana/alloy:v1.15.0` on `127.0.0.1:12345`
- `helsinki.markul.net` was updated again on `2026-04-26` with a dedicated Docker registry compose project at `/home/marat/dev/git/docker/docker-compose/hub-arm`; the new `registry-arm` service is published only through `nginx-proxy` as `https://hub-arm.markul.net/v2/` and intentionally does not expose host port `5000`
- `Markul.Arm` publishing was adjusted on `2026-04-26` so `amd64` images continue to publish to `hub.markul.net` while `arm64` images publish to `hub-arm.markul.net`; both sides now also publish `:dev` alongside the versioned build tags
- The working path is `nginx-proxy` failed-only access log at `/home/marat/dev/git/docker/docker-compose/nginx-proxy/logs/nginx-proxy/failed-requests.log` -> `alloy-logs` file tail at `/var/log/nginx/custom/failed-requests.log` -> `loki.process` parsing and GeoIP enrichment -> local Loki
- Current failed-request labels on `helsinki` include `job="nginx-failed-requests"`, `host`, `method`, `status`, `geoip_country_code`, `geoip_country_name`, `geoip_city_name`, `geoip_subdivision_code`, and `geoip_subdivision_name`; high-cardinality fields such as `client_ip`, `path`, and `user_agent` remain in the log line only
- End-to-end validation on `2026-04-23` confirms a fresh `502` request from `dev.markul.net` to `https://ftp.markul.net/codex-alloy-failed-1776936188` is stored in Loki with `host="ftp.markul.net"`, `method="GET"`, `status="502"`, and GeoIP country `DE/Germany`
- Loki on `helsinki` is now published on host port `3100`, but `ufw` only allows source `46.191.235.87` (`dev.markul.net`) to reach it; validation from `dev` returns `200` on `http://helsinki.markul.net:3100/loki/api/v1/labels`
- `helsinki.markul.net` was re-checked live on `2026-04-28`: `/home/marat/dev/git/docker/docker-compose/helsinki-logging` still matches the tracked repo, `docker compose ps` shows both `loki` and `alloy-logs` up, `alloy-logs` still mounts the dedicated `nginx-proxy` failed-request log read-only, and Alloy metrics show one active tailed file with `loki_write_sent_entries_total=9211`, `loki_write_batch_retries_total=0`, and all `loki_write_dropped_entries_total` reasons at `0`
- The same `2026-04-28` check confirmed Loki `200 OK` on both `/ready` and `/loki/api/v1/labels`, plus a fresh `hub-arm.markul.net` `401` request from `Watchtower (Docker)` was queryable immediately with labels `job="nginx-failed-requests"`, `host="hub-arm.markul.net"`, `method="GET"`, `status="401"`, `source_host="helsinki.markul.net"`, and GeoIP country `DE/Germany`
- Current retained label names observed on `2026-04-28` are `job`, `host`, `method`, `status`, `source_host`, `geoip_country_code`, and `geoip_country_name`; label values for `geoip_city_name` and `geoip_subdivision_name` are currently empty despite the config still declaring them, so city/subdivision enrichment should be treated as unverified until a stream with those labels appears again
- Direct external checks from the current environment on `2026-04-28` also reached `http://helsinki.markul.net:3100/ready` and `/loki/api/v1/labels`, but the host `ufw` rules could not be re-read because `sudo` on `helsinki` required an interactive password
- `helsinki.markul.net` was updated again on `2026-04-28` to suppress self-generated failed-request noise from `45.134.217.102` (`frankfurt.markul.net`) in `nginx-proxy` by splitting the failed-request filter into status and source maps; after `nginx -s reload`, recurring `hub-arm.markul.net` `401` requests from that IP stopped incrementing `failed-requests.log` while unrelated external `503` probes still appended normally
- Grafana on `dev.markul.net` now has one shared dashboard `Failed Requests` at `https://grafana.markul.net/d/failed-requests/failed-requests`, backed by the `Loki source` datasource selector
- The dashboard includes total failed requests, status-over-time, a country map using `geoip_country_code`, country and city counts tables, and recent raw failed requests
- `dev.markul.net` now also has its own local failed-request logging stack under `/home/marat/dev/git/markul/docker/docker-compose/dev.markul.net/logging` with `loki-dev` on `127.0.0.1:3101` / `loki-dev:3100` and `alloy-logs` on `127.0.0.1:12345`
- Grafana datasource `dev.markul.net Loki` (`loki_dev`) points to `http://loki-dev:3100`, and the shared `Failed Requests` dashboard can switch between `helsinki.markul.net` and `dev.markul.net Loki`
- The dev Alloy pipeline drops private client IPs before GeoIP to avoid internal `192.168.*` or Docker bridge traffic being mislabeled as a public country
- [[personal/tech/electricity-consumption|electricity-consumption]] still needs reconciliation with the current `3.5 RUB/kWh` tariff

## Key Decisions

- Keep personal infrastructure tracking in the Obsidian vault rather than creating a separate repo-specific project
- Treat [[personal/tech/infrastructure|Infrastructure]] as the primary machine inventory note
- For the first ingress failed-request implementation, keep ingestion local on `helsinki` with `Loki + Alloy` instead of shipping raw probe logs into `dev` Elasticsearch first
- Tail the dedicated `nginx-proxy` failed-request log instead of Docker JSON logs so successful `2xx/3xx` requests never reach Alloy
- Treat non-`2xx/3xx` responses on the public ingress as suspicious enough for Loki retention, while keeping high-cardinality path and client details out of labels
- Add city and subdivision GeoIP labels for failed-request grouping, but keep postal code, timezone, coordinates, client IP, path, and user agent out of labels unless a later dashboard need justifies the cardinality cost
- Expose Loki over raw `3100/tcp` for Grafana only after restricting `ufw` to the single `dev.markul.net` public IP instead of opening it broadly on the internet

## Blockers

- None currently
