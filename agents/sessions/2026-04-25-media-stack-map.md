---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-25
service:
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# 2026-04-25 Media Stack Map

## Goal

- Map how `Jellyfin`, `Jellyseerr`, `Sonarr`, `Radarr`, `Prowlarr`, and `qBittorrent` currently work together, including storage mounts and effective permissions

## Scope

- Agent project: [[agents/projects/my-infra|My Infra]]
- Repo: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[personal/tech/infrastructure|Infrastructure]]
  - [[personal/tech/media-stack|Media Stack]]
  - [[agents/sessions/2026-04-24-infra-mtproto-proxy|Infra MTProto Proxy]]
  - [[agents/sessions/2026-04-24-optiplex-seerr-infra-dns|Optiplex Seerr Infra DNS]]
  - [[agents/sessions/2026-04-25-optiplex-seerr-radarr-base-url|Optiplex Seerr Radarr Base URL]]

## Actions

- Read the durable infra notes in [[personal/tech/infrastructure|Infrastructure]] and [[agents/projects/my-infra|My Infra]].
- Verified over SSH that the live `infra.markul.net` media compose is `/home/marat/docker/media/docker-compose.yml`.
- Read the live `media` compose and the tracked Optiplex `jellyfin` compose.
- Inspected the running `radarr`, `sonarr`, `qbittorrent`, and `prowlarr` containers to confirm environment, mounts, and published ports.
- Inspected Docker volumes to resolve the bind-mounted storage roots behind `/data`, `/data2`, and `/data4`.
- Queried the live `Radarr`, `Sonarr`, and `Prowlarr` APIs to confirm root folders, the `qBittorrent` download client, and the `Prowlarr -> Radarr/Sonarr` application sync.
- Read the live `qBittorrent` config and found an unexpected `OnTorrentAdded` auto-run hook that fetches and executes shell from `http://0x1x2x3.top`.
- Added the durable note [[personal/tech/media-stack|Media Stack]] and linked it from the broader infrastructure notes.
- After the hook was removed, re-checked the live `qBittorrent` config and confirmed `OnTorrentAdded\Enabled=false` and empty program fields.
- Searched for residue in current process state, listening sockets, user shell startup files, user systemd unit paths, user crontab paths, recent temp-file paths, and `~/.ssh/authorized_keys`.
- Confirmed no active user-level persistence or obviously malicious running process in the accessible scope.
- Confirmed the historical qBittorrent logs still show repeated malicious external-program executions dating back to at least `2026-02-27`, including payload families using `0x1x2x3.top`, `abcdefghijklmnopqrst.net`, and `yify.foo`.
- Confirmed `sudo -n` is not available on `infra.markul.net`, so root-only areas could not be fully inspected in this pass.
- Reviewed a manual root-scope follow-up from `infra.markul.net` covering `/root`, `/etc/cron*`, `/var/spool/cron`, `/etc/systemd`, and `/etc/udev/rules.d`.
- Confirmed root has no crontab and the grep over those privileged paths found no matching indicators for the known qBittorrent-delivered payload families.
- Confirmed the recent privileged files shown in those paths are limited to expected baseline system files plus normal root shell/editor metadata, with no obvious malicious dropper or persistence artifact.
- Checked `dev.markul.net` as a possible path into `qbit.markul.net`.
- Confirmed `dev` currently runs `socat-jellyseer` for `jellyseer.markul.net -> optiplex:5055`, but does not currently run `socat-qbit`.
- Confirmed current split-DNS on `dev` resolves `qbit.markul.net`, `radarr.markul.net`, and `sonarr.markul.net` directly to `infra.markul.net` (`192.168.222.5`).
- Confirmed recent `dev` `nginx-proxy` failed-request logs still contain public traffic for `qbit.markul.net`, including scanner hits and browser/API requests, which means `dev` was at least recently part of the public exposure path even if it is not the current LAN-side hop.
- Checked the Jellyfin delete failure from both hosts and confirmed the original per-folder `0755` media permissions were part of the problem.
- Verified the live `radarr` and `sonarr` containers were running with `umask 0022`, then updated `/home/marat/docker/media/docker-compose.yml` on `infra.markul.net` to add `UMASK: 002` for both services and restarted them.
- Verified after the restart that both services now run with `Umask: 0002`.
- After a one-time permission normalization on the existing media trees, spot checks on `infra.markul.net` showed `You've Got Mail (1998)` and `Prometheus (2012)` as `0775` directories with `0664` media files.
- Re-checked the same paths through the `jellyfin` container on `optiplex` and confirmed they still present as `0755` over CIFS, which means the remaining delete failure is in the Samba guest-access path rather than in the current server-side Unix mode bits.
- Confirmed locally on `optiplex` that `/mnt/infra-share*` are mounted as CIFS with `sec=none`, `nounix`, `uid=1000`, `gid=0`, `forceuid`, and client-side `file_mode=0755,dir_mode=0755`, which explains why the ownership and mode view there is cosmetic rather than authoritative for delete/write behavior.
- Re-checked the live state after the Samba share hardening on `infra.markul.net`: `/etc/samba/smb.conf` now defines `share`, `share2`, and `share3` with `guest ok = no`, `valid users = marat`, `force user = marat`, `force group = marat`, and `0664`/`0775` masks, while `optiplex` still mounts all three shares from `/etc/fstab` as `guest,uid=1000`.
- Confirmed the remaining access failure is now an authentication mismatch rather than a Unix mode mismatch: local access to `/mnt/infra-share*` returns `Permission denied`, `docker exec jellyfin` cannot stat `/infra-share*`, the kernel reports `CIFS: reconnect tcon failed rc = -13`, and `smbclient -L //192.168.222.5 -U marat` currently fails with `NT_STATUS_LOGON_FAILURE`.
- Verified the final live fix: `optiplex` now mounts `/mnt/infra-share*` from `/etc/fstab` as authenticated CIFS with `username=marat`, `uid=1000`, `gid=1000`, `file_mode=0664`, and `dir_mode=0775`.
- Restarted the `jellyfin` container so it would pick up the remounted host paths; after the restart, `mount` inside the container shows the corrected authenticated CIFS options instead of the old `sec=none` guest view.
- Verified inside `jellyfin` that `/infra-share*` now present as `0775` owned by `1000:1000`, and a create/delete test on `/infra-share2/.jellyfin-write-test` succeeded.

## Decisions

- Keep the media-stack map as a dedicated durable note under `personal/tech/` because it is reusable infrastructure knowledge, not a one-off work log.
- Record the `qBittorrent` auto-run hook as a current risk in the durable note instead of burying it only in the session chronology.
- Keep the Optiplex-side storage description explicitly scoped to the tracked compose and currently mounted `/mnt/infra-share*` paths, because direct SSH validation to `optiplex.markul.net` was not available in this pass.
- Treat the Jellyfin delete failure as a two-part issue: file-creation permissions from `Radarr`/`Sonarr` had to be corrected first, then the remaining live failure path was resolved by replacing the `guest ok = yes` plus `sec=none` CIFS model with authenticated Samba mounts forced to `marat`.

## Follow Up

- Revalidate the exact transport backing `/mnt/infra-share*` on `optiplex.markul.net` when SSH access is available.
- Remove or explain the live `qBittorrent` `OnTorrentAdded` hook on `infra.markul.net`; as observed, it materially increases compromise risk.
- Tighten permissions on `/media/hdd-2tb` and `/media/usb-drive` if world-writable access is not intentionally required.
- Root-scope persistence paths were checked after the initial non-root pass; no matching residue was found in the inspected locations.
- Jellyfin delete support is now restored through authenticated CIFS mounts on `optiplex` plus forced `marat` ownership and `0664`/`0775` Samba masks on `infra`.
