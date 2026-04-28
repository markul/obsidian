---
note-type: session
session-date: 2026-04-24
related-project:
  - "[[agents/projects/my-infra|My Infra]]"
related-ticket:
---

# Optiplex Seerr Infra DNS

## Goal

- Find why `seerr` on `optiplex.markul.net` cannot resolve `infra.markul.net` when connecting to Sonarr.

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Host under investigation: `optiplex.markul.net`
- Target hostname: `infra.markul.net`

## Actions

- Verified the current host is `optiplex`.
- Checked the live `seerr` container config with `docker inspect seerr`.
- Checked in-container resolver state with `docker exec seerr cat /etc/resolv.conf`.
- Confirmed host resolution works for `infra.markul.net`.
- Read the tracked compose file at `/home/marat/dev/git/markul/docker/docker-compose/optiplex.markul.net/jellyfin/docker-compose.yml`.
- Confirmed that public resolver `9.9.9.9` returns `NXDOMAIN` for `infra.markul.net`.

## Findings

- `seerr` is configured with an explicit Docker DNS override: `dns: [9.9.9.9]`.
- Inside `seerr`, `/etc/resolv.conf` points Docker's internal stub `127.0.0.11`, which forwards to external server `9.9.9.9`.
- `infra.markul.net` resolves on the host through LAN DNS `192.168.222.100` to `192.168.222.5`.
- `9.9.9.9` does not know the private split-DNS name `infra.markul.net` and returns `NXDOMAIN`.
- The failure is local to the `seerr` container DNS override, not to Sonarr itself.

## Decisions

- Keep the diagnosis focused on resolver path mismatch: host uses LAN DNS, container is pinned to public DNS.

## Validation

- `hostname`
- `docker inspect seerr --format 'Seerr dns={{json .HostConfig.Dns}} networks={{json .NetworkSettings.Networks}}'`
- `docker exec seerr sh -lc 'cat /etc/resolv.conf; getent hosts infra.markul.net || true'`
- `getent hosts infra.markul.net`
- `resolvectl query infra.markul.net`
- `docker run --rm --dns 9.9.9.9 alpine:3.20 sh -lc 'nslookup infra.markul.net 9.9.9.9 2>/dev/null || true'`

## Outcome

- Root cause identified: `seerr` cannot resolve `infra.markul.net` because its compose file hardcodes `9.9.9.9` instead of using the host's LAN resolver.
- Likely fixes are to remove the `dns:` override, replace it with the LAN resolver `192.168.222.100`, or add a static `extra_hosts` entry if the mapping is intentionally local-only.
