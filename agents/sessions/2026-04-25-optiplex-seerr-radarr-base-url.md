---
note-type: session
session-date: 2026-04-25
related-project:
  - "[[agents/projects/my-infra|My Infra]]"
related-ticket:
---

# Optiplex Seerr Radarr Base URL

## Goal

- Find why `seerr` on `optiplex.markul.net` cannot access `radarr.markul.net`.

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Host under investigation: `optiplex.markul.net`
- Container: `seerr`
- Target hostname: `radarr.markul.net`

## Actions

- Checked the tracked compose file at `/home/marat/dev/git/markul/docker/docker-compose/optiplex.markul.net/jellyfin/docker-compose.yml`.
- Inspected the live container with `docker inspect seerr`.
- Verified in-container resolution and HTTP access to `http://radarr.markul.net`.
- Read `/app/config/settings.json` from the live Jellyseerr config volume.
- Compared the saved Radarr `baseUrl` with the live Radarr response.
- Updated the persisted Jellyseerr `settings.json` to clear the stale Radarr `baseUrl`.
- Restarted `seerr` and re-checked container health and logs.

## Findings

- `seerr` can resolve `radarr.markul.net` to `192.168.222.5` through a static `extra_hosts` entry.
- `seerr` can fetch `http://radarr.markul.net` successfully over plain HTTP.
- Jellyseerr had Radarr saved as `hostname=radarr.markul.net`, `port=80`, `useSsl=false`, but `baseUrl=/radarr.markul.net`.
- The live Radarr UI currently serves from `/` with `urlBase: ''`, so Jellyseerr requests under `/radarr.markul.net/...` return `404`.
- Recent `seerr` logs showed repeated `Failed to retrieve profiles: Request failed with status code 404`.

## Decisions

- Treat the current Radarr failure as an application URL-base mismatch, not a DNS or transport problem.
- Keep the static `extra_hosts` workaround in place for now; it is still needed because the container remains pinned to `dns: [9.9.9.9]`.

## Validation

- `docker inspect seerr --format 'dns={{json .HostConfig.Dns}} extra_hosts={{json .HostConfig.ExtraHosts}}'`
- `docker exec seerr sh -lc 'cat /etc/resolv.conf; getent hosts radarr.markul.net || true'`
- `docker exec seerr sh -lc 'wget -S -O - -T 5 http://radarr.markul.net 2>&1 | sed -n "1,40p"'`
- `docker exec seerr sh -lc 'wget -S -O /dev/null -T 5 --header="X-Api-Key: 029717f779c641c3b80701becee0cd1d" http://radarr.markul.net/radarr.markul.net/api/v3/qualityprofile 2>&1 | sed -n "1,40p"'`
- `docker ps --filter name=^/seerr$ --format 'table {{.Names}}\t{{.Status}}'`
- `curl -sS http://localhost:5055/api/v1/status`

## Outcome

- Fixed the live Jellyseerr config by changing the Radarr `baseUrl` from `/radarr.markul.net` to `''` in the persisted `settings.json`.
- Restarted `seerr`; the container is healthy and the post-restart logs no longer show the earlier Radarr `404` profile failures.
