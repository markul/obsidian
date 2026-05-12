---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-24
service:
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# Infra MTProto Proxy

## Goal

- Review the live Docker layout on `infra.markul.net`, find the active compose root, and add an `mtproto-proxy` service on host port `8443`.

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Related durable note: [[personal/tech/infrastructure|Infrastructure]]
- Live host: `infra.markul.net`
- Local repo copy: `/home/marat/dev/git/markul/docker/docker-compose/infra.markul.net`

## Actions

- Verified over SSH that the live compose files for `infra.markul.net` are stored under `/home/marat/docker/`, not under the tracked repo checkout path.
- Enumerated active compose projects on the host: `/home/marat/docker/{media,proxy,registry,ftp,portainer-agent}/docker-compose.yml`.
- Confirmed the host currently runs `radarr`, `sonarr`, `qbittorrent`, `prowlarr`, `wg-easy`, `proxy-tinyproxy-1`, `registry`, and `portainer_edge_agent`.
- Confirmed no existing host port conflict on `8443/tcp` from the running Docker port map.
- Added a tracked compose project at `/home/marat/dev/git/markul/docker/docker-compose/infra.markul.net/mtproto-proxy/docker-compose.yml`.
- Created the matching live compose project at `/home/marat/docker/mtproto-proxy/docker-compose.yml`.
- Started the new service with `docker compose up -d`.
- Verified the live container is `Up` and listening on `0.0.0.0:8443` / `[::]:8443`.
- Captured the initial container output, including the generated proxy secret and the warning that the auto-generated Telegram links default to port `443` even though this deployment is published on `8443`.

## Decisions

- Keep the Telegram proxy as its own top-level compose project on `infra.markul.net` instead of placing it inside the existing `proxy` stack, because the current `proxy` project is a dedicated `tinyproxy` service and there is no shared reverse-proxy pattern on this host.
- Preserve the user's requested runtime behavior as a compose-managed service: `container_name mtproto-proxy`, `restart unless-stopped`, and host mapping `8443:443`.
- Record the repo-to-host drift explicitly: the tracked compose convention is `/home/marat/dev/git/markul/docker/docker-compose/infra.markul.net/...`, while the live host currently deploys from `/home/marat/docker/...`.

## Validation

- `ssh infra.markul.net 'find /home/marat/docker -maxdepth 2 -name docker-compose.yml | sort'`
- `ssh infra.markul.net 'cd /home/marat/docker/mtproto-proxy && docker compose config'`
- `ssh infra.markul.net 'docker ps --filter name=mtproto-proxy --format "table {{.Names}}\t{{.Image}}\t{{.Ports}}\t{{.Status}}"'`
- `ssh infra.markul.net 'ss -ltn sport = :8443'`

## Open Questions

- If this proxy is meant for internet clients rather than LAN-only use, `infra.markul.net` still needs a working external path to `8443/tcp` because the machine inventory note currently records no public entry point for that host.
