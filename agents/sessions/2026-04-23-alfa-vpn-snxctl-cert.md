---
note-type: session
session-date: 2026-04-23
related-project:
related-ticket:
---

# Alfa VPN `snxctl` certificate validation fix

## Goal

Capture the diagnosis and fix for `snxctl connect` returning `Internal IPSec certificate validation failed!`.

## Scope

- Durable troubleshooting note: [[work/alfa-bank/alfa-vpn|Alfa VPN]]
- Local machine: `optiplex`
- VPN client: `snx-rs` / `snxctl`

## Actions

- Checked `snxctl --version`; local version was `5.2.3`.
- Checked system time with `date -u` and `timedatectl`; clock was synchronized.
- Read `~/.config/snx-rs/snx-rs.conf`; it already had `ignore-server-cert=true` and `tunnel-type=ipsec`.
- Ran `snxctl info`; gateway was reachable and advertised internal CA fingerprint `GETS CITE SHAM WALT FORM ROAD FLEW BUFF GROW OS DIAL PRY`.
- Checked running processes and system service state; root `snx-rs -m command -l info` daemon was running under `snx-rs.service`.
- Checked `journalctl -u snx-rs`; recent logs showed previous keepalive and authentication warnings, but no user-service logs.
- Restarted the root daemon with `sudo systemctl restart snx-rs`; user confirmed this fixed `snxctl connect`.

## Decisions

- Treat this error as stale `snx-rs.service` daemon state first; restart the service before changing VPN config.
- Do not rely on `ignore-server-cert=true` for this error because the failure is the internal IPSec certificate fingerprint check, not the outer HTTPS/TLS certificate check.

## Commands

```bash
snxctl --version
date -u
timedatectl
snxctl info
snxctl status
systemctl status snx-rs --no-pager
journalctl -u snx-rs -n 160 --no-pager
sudo systemctl restart snx-rs
snxctl connect
```

## Outcome

`sudo systemctl restart snx-rs` fixed the connection. The durable quick fix is documented in [[work/alfa-bank/alfa-vpn|Alfa VPN]].
