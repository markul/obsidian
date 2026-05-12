---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-25
service:
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# Markul Local Systemd Resolved

## Goal

- Find why `optiplex`, `dev.markul.net`, and `infra.markul.net` cannot resolve new router DNS names under `markul.local` such as `qbit.markul.local`.

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Hosts under investigation: `optiplex`, `dev.markul.net`, `infra.markul.net`
- Example failing name: `qbit.markul.local`

## Actions

- Checked `resolvectl status`, `resolvectl query`, `getent hosts`, and `/etc/resolv.conf` on `optiplex`.
- Repeated the same checks over SSH on `dev.markul.net` and `infra.markul.net`.
- Queried the router DNS directly at `192.168.222.100` for `qbit.markul.local`.
- Compared resolution for `qbit.markul.net` versus `qbit.markul.local`.

## Findings

- All three hosts use `systemd-resolved` stub mode with router DNS `192.168.222.100`.
- `qbit.markul.net` resolves normally on all three hosts.
- `qbit.markul.local` fails on all three hosts with `resolve call failed: No appropriate name servers or networks for name found`.
- A direct DNS query to `192.168.222.100` for `qbit.markul.local` returns `192.168.222.5`, so the router record exists and answers correctly.
- The failure is local to Linux resolver behavior for `.local`, not to the router DNS record itself.

## Decisions

- Treat `.local` as the wrong long-term zone for router-managed unicast DNS on these Linux hosts.
- Prefer a normal unicast DNS zone such as `markul.net` or `home.arpa` for new static router records.

## Validation

- `resolvectl query qbit.markul.net`
- `resolvectl query qbit.markul.local`
- `getent hosts qbit.markul.local`
- `python3 - <<'PY' ... socket.getaddrinfo(...)`
- `dig @192.168.222.100 qbit.markul.local +short`

## Outcome

- Root cause identified: `systemd-resolved` is not forwarding `.local` names to the router DNS by default, so new router records under `markul.local` fail on `optiplex`, `dev`, and `infra` even though the router serves them.
- Clean fix: stop using `.local` for router unicast DNS.
- Host-side workaround if `.local` must remain: explicitly route `~markul.local` to DNS server `192.168.222.100` on each affected host.
