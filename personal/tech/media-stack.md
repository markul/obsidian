# Media Stack

## Related Notes

- [[personal/tech/index|Tech]]
- [[personal/tech/Infrastructure|Infrastructure]]
- [[agents/projects/my-infra|My Infra]]

## Overview

- `infra.markul.net` is the acquisition and library-management host: `prowlarr`, `radarr`, `sonarr`, `qbittorrent`
- `optiplex.markul.net` is the playback and request host: `jellyfin`, `seerr`
- The current live media compose on `infra.markul.net` runs from `/home/marat/docker/media/docker-compose.yml`
- The tracked Optiplex compose is `/home/marat/dev/git/markul/docker/docker-compose/optiplex.markul.net/jellyfin/docker-compose.yml`

## Service Roles

- `Prowlarr` is the indexer manager; it currently has `RuTracker.org` enabled and pushes indexer definitions to both `Radarr` and `Sonarr` with `fullSync`
- `Radarr` manages movies and sends torrent jobs to `qBittorrent` with category `movies`
- `Sonarr` manages TV shows and sends torrent jobs to `qBittorrent` with category `tv`
- `qBittorrent` is the only configured download client for both `Radarr` and `Sonarr`
- `Jellyseerr` is the request UI on `optiplex.markul.net`; it talks to `Radarr` and `Sonarr` over LAN HTTP
- `Jellyfin` reads the shared libraries from mounts on `optiplex.markul.net` and serves playback

## Request And Download Flow

1. A request is made in `Jellyseerr`.
2. `Jellyseerr` forwards the request to `Radarr` or `Sonarr` on `infra.markul.net`.
3. `Radarr` or `Sonarr` uses `Prowlarr`-managed indexers to find releases.
4. `Radarr` or `Sonarr` submits the selected torrent to `qBittorrent` on the shared Docker network using host `qbittorrent:8080`.
5. `qBittorrent` downloads into the shared media storage exposed to the whole `media` compose.
6. `Radarr` or `Sonarr` imports from the downloaded files into one of the library roots under `/data`, `/data2`, or `/data4`.
7. `Jellyfin` reads those same libraries remotely through the `/mnt/infra-share*` mounts on `optiplex.markul.net`.

## Storage Layout

### Infra Side

- `media_data` -> bind mount to `/media/hdd-drive/network-share/` -> exposed inside containers as `/data`
- `media_data2` -> bind mount to `/media/hdd-2tb/` -> exposed as `/data2`
- `media_data4` -> bind mount to `/media/usb-drive/` -> exposed as `/data4`
- `Radarr` root folders:
  - `/data/media/movies`
  - `/data2/media/movies`
  - `/data4/media/movies`
- `Sonarr` root folders:
  - `/data/media/tv`
  - `/data2/media/tv`
  - `/data4/media/tv`
- `qBittorrent` saves torrents under `/data2/torrents` with temp data in `/data2/torrents/temp`
- App configs live in Docker-managed volumes:
  - `media_radarr-config`
  - `media_sonarr-config`
  - `media_qbit-config`
  - `media_prowlarr-config`

### Optiplex Side

- `Jellyfin` mounts:
  - `/mnt/infra-share` -> `/infra-share`
  - `/mnt/infra-share2` -> `/infra-share2`
  - `/mnt/infra-share3` -> `/infra-share3`
- The local tracked compose indicates `Jellyfin` is reading remote library mounts instead of managing local media storage directly
- The live `optiplex` host mounts those paths over authenticated CIFS from `infra.markul.net` (`//192.168.222.5/share*`) with `username=marat`, `uid=1000`, `gid=1000`, `file_mode=0664`, and `dir_mode=0775`

## Permissions Model

- `marat` on `infra.markul.net` is `uid=1000`, `gid=1000`
- `Radarr`, `Sonarr`, `Prowlarr`, and `qBittorrent` all run LinuxServer images with `PUID=1000` and `PGID=1000`
- This means the media apps read and write as the host user `marat`, not as root
- The shared storage roots currently have these top-level permissions on `infra.markul.net`:
  - `/media/hdd-drive/network-share` -> `drwxrwxr-x` (`775`) owner `marat:marat`
  - `/media/hdd-2tb` -> `drwxrwxrwx` (`777`) owner `marat:marat`
  - `/media/usb-drive` -> `drwxrwxrwx` (`777`) owner `marat:marat`
- On the local machine, the `Jellyfin` mountpoints currently appear as:
  - `/mnt/infra-share` -> `drwxrwxr-x` (`775`) owner `marat:marat`
  - `/mnt/infra-share2` -> `drwxrwxr-x` (`775`) owner `marat:marat`
  - `/mnt/infra-share3` -> `drwxrwxr-x` (`775`) owner `marat:marat`
- `Jellyfin` is configured with `PUID=1000` and `PGID=1000`, so it also accesses files as the same host UID/GID model
- `Jellyseerr` has no media mounts; it only keeps app state in `jellyseerr-config`
- `Radarr` and `Sonarr` run with `UMASK=002`, so new imports land as `0775` directories and `0664` files
- Existing media under `/media/hdd-2tb/media` and `/media/usb-drive/media` is normalized to `0775` directories and `0664` files
- Samba now exports `share`, `share2`, and `share3` with `guest ok = no`, `read only = no`, `valid users = marat`, `force user = marat`, `force group = marat`, `create mask = 0664`, and `directory mask = 0775`
- `Optiplex` mounts those shares from `/etc/fstab` with credentials-backed CIFS, and the live mount options show `username=marat`, `uid=1000`, `gid=1000`, `file_mode=0664`, and `dir_mode=0775`
- Inside `jellyfin`, `/infra-share*` present as `0775` owned by `1000:1000`, and the service can create and delete files on the mounted libraries

## Networking

- `infra.markul.net` publishes the app UIs on host ports `7878`, `8989`, `9696`, and `8080`
- The `media` compose joins the external Docker network `nginx-proxy`
- `VIRTUAL_HOST` is set for `qbit.markul.net`, `radarr.markul.net`, `sonarr.markul.net`, and `prowlarr.markul.net`, so the stack is intended to sit behind the shared `nginx-proxy`
- `dev.markul.net` does not currently run `socat-qbit`, and current split-DNS on `dev` resolves `qbit.markul.net`, `radarr.markul.net`, and `sonarr.markul.net` directly to `infra.markul.net` (`192.168.222.5`) rather than to `dev`
- `dev.markul.net` still runs public reverse-proxying for `jellyseer.markul.net` through `socat-jellyseer -> optiplex:5055`
- `Jellyseerr` currently uses public DNS `9.9.9.9` plus static `extra_hosts` entries for `infra.markul.net`, `radarr.markul.net`, and `sonarr.markul.net`
- That DNS override is a workaround and means future internal names will fail unless they are also hardcoded or the resolver path is corrected

## Current Risks

- Two storage roots are world-writable (`777`), which reduces friction for media imports but also broadens the blast radius if a container or host process is compromised
- `Jellyseerr` depends on static host mappings because of the pinned public DNS override instead of using the LAN resolver directly
