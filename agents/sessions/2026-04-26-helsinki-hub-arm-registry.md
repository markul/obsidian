---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-26
related-project:
  - "[[agents/projects/my-infra|My Infra]]"
related-ticket:
---

# Helsinki Hub-Arm Registry

## Goal

- Add a new Docker registry on `helsinki.markul.net` that is reachable only through `https://hub-arm.markul.net/v2/`, without publishing host port `5000`.

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Related durable notes: [[personal/tech/Infrastructure|Infrastructure]], [[personal/tech/software|Software]]
- Source host for the sample registry: `infra.markul.net`
- Live target host: `helsinki.markul.net`
- Live compose path: `/home/marat/dev/git/docker/docker-compose/hub-arm`

## Actions

- Reviewed the tracked `hub.markul.net` sample under `/home/marat/dev/git/markul/docker/docker-compose/dev.markul.net/socat/docker-compose.yml` and confirmed it is a reverse-proxy `socat` front, not the registry implementation itself.
- Checked the live Helsinki Docker layout and confirmed the active compose root is `/home/marat/dev/git/docker/docker-compose/` with an external `nginx-proxy` Docker network already in use.
- Inspected the live registry sample on `infra.markul.net` at `/home/marat/docker/registry/docker-compose.yml` and reused its `registry:2` plus `htpasswd` authentication shape as the implementation baseline.
- Created `/home/marat/dev/git/docker/docker-compose/hub-arm/docker-compose.yml` on `helsinki.markul.net` with a `registry-arm` service using `registry:2`, `REGISTRY_STORAGE_DELETE_ENABLED=true`, and `REGISTRY_AUTH=htpasswd`.
- Mounted `./auth:/auth:ro`, created a named Docker volume `registry_arm_data`, and attached the container only to the external `nginx-proxy` network.
- Kept the registry private to Docker networking by omitting any host `ports:` mapping and exposing the service to `nginx-proxy` via `VIRTUAL_HOST=hub-arm.markul.net`, `VIRTUAL_PORT=5000`, and matching `LETSENCRYPT_*` variables.
- Copied the existing `htpasswd` file from `infra.markul.net:/home/marat/docker/registry/auth/registry.password` into `helsinki.markul.net:/home/marat/dev/git/docker/docker-compose/hub-arm/auth/registry.password`, preserving the current `admin` login identity.
- Started the new stack with `docker compose up -d` on `helsinki.markul.net`.
- Verified that `nginx-proxy` regenerated its config for `hub-arm.markul.net` and that `nginx-proxy-letsencrypt` issued a fresh certificate for the new hostname.
- Validated the registry endpoint: `https://hub-arm.markul.net/v2/` now returns `401 Unauthorized` with `docker-distribution-api-version: registry/2.0` and `www-authenticate: Basic realm="Registry"`, which is the expected healthy response for an authenticated Docker registry before login.

## Decisions

- Use the live `infra.markul.net` registry compose as the primary sample rather than the tracked `socat-registry` helper, because the Helsinki task required a full registry instance, not just a TCP forwarder.
- Keep the Helsinki registry behind the existing `jwilder/nginx-proxy` and `letsencrypt-nginx-proxy-companion` stack so the public interface is HTTPS on `hub-arm.markul.net` instead of a raw published `5000/tcp`.
- Reuse the current `htpasswd` file from `infra.markul.net` so `hub-arm.markul.net` initially matches the existing registry authentication setup.

## Validation

- `ssh helsinki.markul.net 'cd /home/marat/dev/git/docker/docker-compose/hub-arm && docker compose up -d'`
- `ssh helsinki.markul.net 'docker ps --filter name=registry-arm --format "table {{.Names}}\t{{.Image}}\t{{.Status}}"'`
- `ssh helsinki.markul.net 'docker inspect registry-arm --format "{{json .NetworkSettings.Networks}}"'`
- `ssh helsinki.markul.net 'curl -skI https://hub-arm.markul.net/v2/'`
- `ssh helsinki.markul.net 'curl -sk https://hub-arm.markul.net/v2/'`

## Open Questions

- The copied `htpasswd` file preserves the existing `admin` username but not a recoverable plaintext password; if the password is unknown in practice, the next step is to rotate the registry credentials and redistribute them to clients.
- The registry logs warn that `REGISTRY_HTTP_SECRET` is unset; that is acceptable for a single instance, but a fixed secret should be added before introducing any multi-instance or load-balanced registry path.
