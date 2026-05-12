---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-21
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# 2026-04-21 my-infra proxmox node-exporter readiness and install

## Goal

- Verify whether the actual Proxmox hypervisor can run `node_exporter` and install it if the host is ready

## Scope

- Agent project: [[agents/projects/my-infra|My Infra]]
- Repo: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[personal/tech/infrastructure|Infrastructure]]
  - [[personal/tech/hardware|hardware]]
  - [[daily/2026-04-21]]

## Actions

- Re-read the durable infra notes to confirm the target hypervisor:
  - `proxmox.markul.net`
  - LAN IP `192.168.222.3`
- Confirmed that `proxmox.markul.net` resolves from this workstation to the public IP `46.191.235.87`, which caused an SSH host-key mismatch against the expected internal host
- Verified the Proxmox management endpoint and SSH listener on the LAN IP before retrying access
- Retried authenticated access after the SSH key was added for `root@192.168.222.3`
- Checked the live host state on the actual hypervisor:
  - hostname `proxmox`
  - `Debian GNU/Linux 12 (bookworm)`
  - `x86_64`
  - `systemd 252`
  - `pve-manager/8.4.5`
- Checked `node_exporter` readiness:
  - no `node_exporter` binary in `PATH`
  - no `node_exporter.service`
  - no process already running
  - port `9100` not listening
- Checked basic capacity:
  - root filesystem `94G` total, `77G` available
  - memory `31 GiB` total, about `10 GiB` available
  - system state `running`
- Verified outbound access from the hypervisor to the upstream release location with HTTP `200` for `https://github.com/prometheus/node_exporter/releases/latest`
- Checked Proxmox firewall state: `disabled/running`
- Confirmed the current upstream release from the official repository: `node_exporter` `1.11.1`
- Installed `node_exporter` on `proxmox`:
  - created dedicated system user and group `node_exporter`
  - installed the upstream `linux-amd64` binary to `/usr/local/bin/node_exporter`
  - created `/etc/systemd/system/node_exporter.service`
  - enabled and started the service with systemd
- Verified the live install state on the hypervisor:
  - `node_exporter.service` is `active (running)`
  - listener is present on `*:9100`
  - local metrics endpoint responds on `http://127.0.0.1:9100/metrics`
  - exported build metric reports version `1.11.1`
- Re-checked the live Proxmox storage layout after install:
  - `nvme0n1` `~1 TB` backs `local` root storage and the `local-lvm` thin pool
  - `sdc1` `~500 GB` ext4 is mounted as `/mnt/pve/hdd500`
  - `sdb` `~1 TB` backs the `hdd1000` LVM-thin pool
  - `sdd1` `~2 TB` backs `usb-drive2000`
  - `pvesm status` currently reports:
    - `local` `~98 GB` total, `~80 GB` available
    - `local-lvm` `~833 GB` total, `~529 GB` available
    - `hdd500` `~480 GB` total, `~449 GB` available
    - `hdd1000` `~957 GB` total, `~130 GB` available
    - `usb-drive2000` `~1.95 TB` total, `~13.6 GB` available
- Verified the DNS/SSH fix for the hypervisor hostname:
  - after removing the public `proxmox.markul.net` record and adding a local router DNS record, this workstation now resolves `proxmox.markul.net` to `192.168.222.3`
  - removed the stale `proxmox.markul.net` entry from `~/.ssh/known_hosts`
  - re-added the current internal host key through a successful SSH connection
  - `ssh root@proxmox.markul.net` now lands on the LAN Proxmox host cleanly
- Checked the Prometheus setup on `dev.markul.net` and added Proxmox scraping:
  - Prometheus runs as Docker container `monitoring_prometheus`
  - live config is bind-mounted from `/home/marat/dev/git/markul/docker/docker-compose/dev.markul.net/monitoring/prometheus/prometheus.yml`
  - after flushing DNS caches on `dev`, `proxmox.markul.net` resolved correctly to `192.168.222.3`
  - verified from inside the Prometheus container that `http://proxmox.markul.net:9100/metrics` responds
  - updated the scrape target to `proxmox.markul.net:9100`
  - copied the updated config to `dev`, validated it with `promtool`, and restarted only `monitoring_prometheus`
  - confirmed the running container now carries the `proxmox.markul.net:9100` target in `/etc/prometheus/prometheus.yml`
- Analyzed `socat-proxmox` on `dev.markul.net`:
  - the container runs `socat TCP-LISTEN:8006,fork,reuseaddr TCP:192.168.222.3:8006`
  - it is not published on host ports; it is only exposed on the Docker `nginx-proxy` network
  - `nginx-proxy` has a virtual host for `proxmox.markul.net` with a Let's Encrypt certificate and proxies `https://proxmox.markul.net` to the socat container on port `8006`
  - the path is therefore: client HTTPS -> `dev` `nginx-proxy` -> `socat-proxmox` TCP forward -> `https://192.168.222.3:8006`
  - because local DNS now resolves `proxmox.markul.net` directly to `192.168.222.3`, normal LAN access bypasses this container entirely
  - this reverse-proxy path now only makes sense if you still want `dev` to front Proxmox for public HTTPS access
  - the current certificate on `dev` for `proxmox.markul.net` is valid until `2026-06-17`, but renewal will likely stop working if the public DNS record remains removed
- Removed the obsolete `dev`-side Proxmox reverse-proxy path:
  - deleted the `socat-proxmox` service from `/home/marat/dev/git/markul/docker/docker-compose/dev.markul.net/socat/docker-compose.yml`
  - copied the updated compose file to `dev`
  - force-removed only the running `socat-proxmox` container on `dev`
  - verified `docker compose ps` no longer lists `socat-proxmox`
  - verified `nginx-proxy` no longer serves a Proxmox vhost; a local HTTPS request to `nginx-proxy` with `Host: proxmox.markul.net` now falls through to the default `503`
- Installed `node_exporter` on `helsinki.markul.net`:
  - accessed the host as `marat` and used `sudo`
  - host details: `Ubuntu 24.04.4`, `aarch64`, `7.5 GiB` RAM, `~52 GiB` free on `/`
  - installed the upstream `linux-arm64` binary to `/usr/local/bin/node_exporter`
  - created `/etc/systemd/system/node_exporter.service`
  - enabled and started the service with systemd
  - verified `node_exporter.service` is `active (running)`, `*:9100` is listening, and the local metrics endpoint reports version `1.11.1`
- Installed `node_exporter` on `frankfurt.markul.net`:
  - refreshed the stale Frankfurt SSH host key locally and later used the newly added `root` key
  - host details: `Ubuntu 22.04.5`, `x86_64`, `957 MiB` RAM, `~12 GiB` free on `/`
  - installed the upstream `linux-amd64` binary to `/usr/local/bin/node_exporter`
  - created the dedicated `node_exporter` system user and systemd unit
  - the first remote wrapper left trailing garbage in the unit file; replaced `/etc/systemd/system/node_exporter.service` with the clean unit and reloaded systemd
  - verified `node_exporter.service` is `active (running)`, `*:9100` is listening, and the local metrics endpoint reports version `1.11.1`
- Verified Prometheus reachability from `dev.markul.net` and completed VPS scrape wiring:
  - `frankfurt.markul.net:9100` was immediately reachable from inside the Prometheus container by hostname
  - `helsinki.markul.net:9100` initially timed out even though `node_exporter` was listening; the blocker was `ufw`, which only allowed `22/tcp`
  - opened `9100/tcp` on `helsinki`
  - re-verified from `dev` that both `http://helsinki.markul.net:9100/metrics` and `http://frankfurt.markul.net:9100/metrics` return `node_exporter_build_info`
  - Prometheus on `dev` already had both hostname targets in `prometheus.yml`; synced the current config, restarted `monitoring_prometheus`, validated it with `promtool`, and confirmed the running config carries:
    - `helsinki.markul.net:9100`
    - `frankfurt.markul.net:9100`
- Checked recent `nginx-proxy` logs on the public ingress hosts for exploit-style probes:
  - `dev.markul.net`: no clear vulnerability-probe requests were visible in the recent sampled `nginx-proxy` stdout; the pattern matches in that window were false positives from normal Grafana API calls and internal service traffic
  - `helsinki.markul.net`: active opportunistic probing is present in the recent `nginx-proxy` logs, including:
    - `134.209.223.209` probing `ftp.markul.net` for `//xmlrpc.php?rsd` and `//wordpress/wp-includes/wlwmanifest.xml`
    - repeated `GET /.env` requests from `78.153.140.149` and `124.198.131.162`
    - `GET /.git/HEAD` from `77.83.39.94`
    - repeated `GET /.git/config` from `170.64.232.230`
    - `GET /boaform/admin/formLogin?...` from `38.196.90.139`
    - `GET /HNAP1` from `157.230.21.134`
    - `GET /solr/admin/info/system` and `GET /solr/admin/cores?...` from `209.38.211.125`
    - `GET /cgi-bin/authLogin.cgi` from `104.248.27.192`
  - most of these requests were handled by the default proxy response path and returned `400`, `502`, or `503`, which is consistent with generic internet scanning rather than successful exploitation
- Analyzed how to visualize probe traffic in Grafana by source country:
  - Grafana on `dev.markul.net` already has a working `Elasticsearch` datasource in addition to `Prometheus`
  - the current Elasticsearch `logstash-*` indices do not contain the `nginx-proxy` probe events seen in raw `docker logs`
  - both `dev` and `helsinki` use Docker `json-file` logging for `nginx-proxy`, so those access logs can be shipped directly from the host log files
  - tested an Elasticsearch ingest pipeline on `dev` with a real `helsinki` probe line and confirmed it can parse the access-log format and enrich `client_ip` with `geoip.country_name` and `geoip.location`
  - prepared the implementation plan before any config changes:
    - create a dedicated ingest pipeline on `dev` Elasticsearch for `nginx-proxy` access logs with GeoIP enrichment
    - add a lightweight shipper, starting with `helsinki`, to send only `nginx-proxy` events into a dedicated index such as `nginx-probe-*`
    - keep probe filtering focused on the currently observed exploit-style paths first
    - build a Grafana dashboard in `Main` with a world map, top countries, top probe paths, and recent events
    - validate end to end on `helsinki` before expanding the same path to other public hosts
- Implemented the `helsinki` logging path directly on the host:
  - created `/home/marat/dev/compose/helsinki-logging` from the local repo at `/home/marat/dev/git/markul/docker/docker-compose/helsinki.markul.net/logging`
  - deployed `grafana/loki:3.7.0` bound to `127.0.0.1:3100` and `grafana/alloy:v1.15.0` bound to `127.0.0.1:12345`
  - downloaded the `ip66.mmdb` GeoIP database into `geoip/ip66.mmdb`
  - switched from the initial `loki.source.docker` approach to `loki.source.file` reading `/var/lib/docker/containers/<container-id>/<container-id>-json.log` because `source.file` supports `tail_from_end = true` and avoids replaying the full historical Docker log
  - mounted `/var/lib/docker/containers` read-only into the Alloy container so it can tail the host `json-file` log directly
- Built and validated the working `Alloy` pipeline for `nginx-proxy` probes on `helsinki`:
  - parse Docker JSON log entries with `stage.docker`, then switch the active line to the extracted `output`
  - strip ANSI color codes, parse the `nginx-proxy` access-log format, and keep only access-log lines
  - classify suspicious requests into `probe_class` values `env`, `git`, `wordpress`, `hnap`, `solr`, `boaform`, `cgi-auth`, plus a generic `http-error` path for `4xx/5xx`
  - enrich `client_ip` with GeoIP country data from `ip66.mmdb`
  - write labeled probe events into local Loki under `job="nginx-proxy-probes"`
- End-to-end validation on `helsinki`:
  - a local probe request for `GET /.env` is present in Loki under `probe_class="env"`
  - repeated local probe requests for `GET /boaform/admin/formLogin` are present in Loki under `probe_class="boaform"`
  - the log line remains readable in Loki instead of being packed into JSON for new events
  - local self-tests originate from `172.20.0.1`, so they do not produce GeoIP country labels; real internet probe traffic should populate those fields because the enrichment already runs on `client_ip`
- Exposed Loki for Grafana reachability without opening it broadly:
  - changed the host port mapping from `127.0.0.1:3100:3100` to `3100:3100`
  - resolved `dev.markul.net` to public IP `46.191.235.87`
  - added `ufw allow from 46.191.235.87 to any port 3100 proto tcp` on `helsinki`
  - confirmed `helsinki` is listening on `0.0.0.0:3100`
  - confirmed from `dev.markul.net` that `http://helsinki.markul.net:3100/loki/api/v1/labels` returns `200`
- Noted one cleanup artifact from earlier failed iterations:
  - there is still one older test event in Loki stored in the previous packed JSON shape from before the final pipeline settled; it will age out under the current retention period and does not affect new ingestion
- Investigated why the new Grafana Loki datasource on `dev` still did not show source-country fields:
  - verified the datasource can reach `helsinki` Loki, but the available labels still only expose the base stream labels, not `geoip_country_*`
  - instrumented the `Alloy` pipeline with `loki.echo` and confirmed the current processed entries only carry the original static labels; parsed access-log fields and GeoIP metadata never appear in the final entry
  - several parsing attempts were tested against the colored Docker `json-file` lines:
    - `stage.regex` with progressively looser patterns
    - `stage.pattern`
    - explicit `stage.json` plus `stage.output`
    - explicit structured-metadata mapping and `allow_structured_metadata: true` in Loki
  - none of those attempts produced `vhost`, `status`, or `geoip_country_*` on the emitted entries, which means the access-log parse is still not matching the real line shape as Alloy sees it
  - restored `helsinki` back to the minimal working collector state so the raw `nginx-proxy` access logs continue flowing into Loki instead of leaving the source half-broken
- Recreated the `helsinki` `nginx-proxy` stack from local Docker Compose because Portainer Edge Agent could not currently manage the host:
  - confirmed the running Portainer-created proxy containers used volumes `nginx-certs`, `nginx_vhost.d`, and `nginx_html` on the external `nginx-proxy` network
  - updated the local `docker-compose/helsinki.markul.net/nginx-proxy/docker-compose.yml` to reuse those external volumes instead of creating new compose-scoped volumes
  - synced the compose directory to `/home/marat/dev/git/docker/docker-compose/nginx-proxy` on `helsinki`
  - removed only the old `nginx-proxy` and `nginx-proxy-letsencrypt` containers and recreated them with `docker compose up -d`
  - verified the new containers now belong to compose project `nginx-proxy`, retain the existing cert/vhost/html volumes, and load `config/custom-settings.conf`
  - validated `nginx -t` successfully; the only warnings were existing OCSP stapling warnings for the current certificates
  - verified `https://drone.markul.net` responds from `dev` via `65.108.89.190`; `https://ftp.markul.net` currently returns `502`, so that backend may need a separate check
- Implemented and validated the proxy-side failed-request log on `helsinki`:
  - added a persistent bind mount from the local compose path `logs/nginx-proxy` to `/var/log/nginx/custom` inside `nginx-proxy`
  - added `failed_only` log format and `map $status $log_failed` in `config/custom-settings.conf` so `2xx` and `3xx` responses are excluded
  - tested an initial HTTP-level `access_log` directive and confirmed it did not capture vhost failures because the generated `jwilder/nginx-proxy` server blocks define their own `access_log`
  - copied the stock `/app/nginx.tmpl` into `templates/nginx.tmpl`, patched the template `access_log` block to add `/var/log/nginx/custom/failed-requests.log failed_only if=$log_failed`, and mounted it back into the container
  - recreated only the `nginx-proxy` container and confirmed generated vhost config now contains the failed-only access log directive
  - validated filtering from `dev.markul.net`: `https://drone.markul.net/` returned `303` and was not logged; `https://ftp.markul.net/` returned `502` and produced exactly one line in `failed-requests.log`
  - next logging step is to update `alloy-logs` to tail `/var/log/nginx/custom/failed-requests.log` instead of the full Docker JSON log

## Decisions

- Treat the hypervisor as ready for a standard `node_exporter` install from the upstream Linux `amd64` release tarball
- Install the exporter as a dedicated systemd service user instead of reusing the article's earlier `prometheus` user assumption
- Use `proxmox.markul.net` for normal host administration again now that local DNS resolves it to `192.168.222.3` and the SSH host key was refreshed
- Keep the default exporter listener on port `9100`; remote scraping still depends on the eventual scraper path and any firewall decision

## Follow Up

- Decide which Prometheus or other scraper host should collect from `proxmox:9100`, then add the matching firewall rule only if needed
- Decide whether to clean up the stale `proxmox.markul.net` certificate material left on `dev` by `nginx-proxy-letsencrypt`; it is no longer used after the proxy path removal
- Decide whether the public `helsinki` ingress should add extra filtering or monitoring for repeated generic vulnerability probes; the current logs show the expected internet background noise but also identify concrete source IPs and paths if blocking becomes worthwhile
- Review whether `usb-drive2000` staying at `99.3%` usage is acceptable for the current VM `102` data placement
- Add `helsinki` local Loki as a Grafana datasource on `dev.markul.net` or otherwise bridge it into the main Grafana instance
- Fix the `Alloy` access-log parser on `helsinki` before relying on GeoIP or status-based dashboards; current state keeps the log feed working but does not emit `geoip_country_*`
- Build the first Grafana dashboard around the raw `nginx-proxy` Loki feed first, then add country/status panels once parsed fields are actually present
- Expand the same `Alloy` path to other public ingress hosts only after the `helsinki` dashboard and query shape are stable
