# Infrastructure

## Related Notes

- [[personal/index|Personal]]
- [[agents/projects/my-infra|My Infra]]
- [[personal/tech/media-stack|Media Stack]]
- [[personal/tech/software|Software]]

## Overview

- Home infrastructure is centered on `proxmox.markul.net`, which hosts the main internal VMs: `dev`, `alfa`, and `infra`
- `raspberrypi.markul.net` is a lightweight always-on ARM node for small Docker services and monitoring/utility workloads
- `optiplex.markul.net` is a separate physical machine for newer x86 workloads, local Kubernetes experiments, media, and CI runners
- `frankfurt.markul.net` and `helsinki.markul.net` are public VPS nodes for VPN, proxying, Portainer, and CI edge services

## Network

### Core Endpoints

| Host | Type | Primary role | Internal IP | Public access |
| --- | --- | --- | --- | --- |
| `proxmox.markul.net` | Physical | Hypervisor | `192.168.222.3` | None noted |
| `raspberrypi.markul.net` | Physical | Lightweight ARM utility/app node | `192.168.222.40` | None noted |
| `dev.markul.net` | VM `100` | Main Docker app host | `192.168.222.8` | `46.191.235.87:55755` |
| `alfa.markul.net` | VM `101` | Kubernetes dev/test VM | `192.168.222.16` | `46.191.235.87:55955` intended / unverified |
| `infra.markul.net` | VM `102` | Media and utility host | `192.168.222.5` | None noted |
| `optiplex.markul.net` | Physical | Secondary Docker / Minikube / media host | `192.168.222.2` | `46.191.235.87:55855` |
| `frankfurt.markul.net` | VPS | Lightweight VPN / edge node | N/A | Hostname `739712.cloud4box.ru` |
| `helsinki.markul.net` | VPS | Public reverse proxy / CI / VPN node | N/A | IP `65.108.89.190` |

### Public Entry Points

- `46.191.235.87:55755` -> `dev.markul.net`
- `46.191.235.87:55855` -> `optiplex.markul.net`
- `46.191.235.87:55955` -> `alfa.markul.net` intended / unverified
- `739712.cloud4box.ru` -> `frankfurt.markul.net`
- `65.108.89.190` -> `helsinki.markul.net`

## Service Placement

### Virtualization And Orchestration

- `proxmox.markul.net` is the main hypervisor and currently runs VMs `100`, `101`, and `102`
- `alfa.markul.net` runs `minikube` with one known app workload: `ufr-kpnsb-limit-service`
- `optiplex.markul.net` also runs `minikube`, but currently only system/control-plane pods were detected
- `raspberrypi.markul.net` runs Docker and no Kubernetes tooling was detected

### App, Data, And Integration Services

- `dev.markul.net` is the primary application host for the Markul stack
- Main app containers on `dev.markul.net`: `markul.engine`, `markul.api`, `markul.media`, `markul.web`, `markul.canary`, `markul.instagram`, `identity`, `identity-api`, `sts`
- Data and messaging on `dev.markul.net`: `sql-server`, `postgres`, `pgadmin`, `consul`, `kafka`, `kafka-ui`, `redpanda`
- Observability on `dev.markul.net`: `grafana`, `grafana_renderer`, `monitoring_prometheus`, `monitoring_cadvisor`, `monitoring_node_exporter`, `kibana`, `elasticsearch`, `loki-dev`, `alloy-logs`; Grafana reads both the local dev Loki datasource `dev.markul.net Loki` and the remote `helsinki.markul.net` Loki datasource
- `raspberrypi.markul.net` runs a smaller app-side Docker set including `markul.client.dev` and `markul.auth.proxy.dev`

### Edge, Proxy, And CI Services

- `dev.markul.net` also carries reverse proxying, forwarding, and automation: `nginx-proxy`, `nginx-proxy-letsencrypt`, `watchtower`, `portainer`, multiple `socat-*` containers, `drone-runner`, `woodpecker-agent-woodpecker-agent-1`, `ut99-server`
- `helsinki.markul.net` is the main public edge/CI VPS: `nginx-proxy`, `nginx-proxy-letsencrypt`, `drone-server`, `drone-runner`, `wireguard`, `portainer_edge_agent`, and the HTTPS-only `hub-arm.markul.net` Docker registry
- `frankfurt.markul.net` is a minimal public node for `amnezia-awg` and `portainer_edge_agent`

### Media And Utility Services

- `infra.markul.net` is the main media/download VM: `sonarr`, `radarr`, `prowlarr`, `qbittorrent`
- `infra.markul.net` also carries utility services: `wg-easy`, `proxy-tinyproxy-1`, `registry`, `portainer_edge_agent`, `mtproto-proxy`
- `optiplex.markul.net` also hosts media and supporting utilities: `jellyfin`, `seerr`, `portainer_edge_agent`, `monitoring_cadvisor`, `monitoring_node_exporter`, `drone-runner`, `vagrant-boxes`
- `raspberrypi.markul.net` also runs `mediamtx`, `watchtower`, `portainer_edge_agent`, `monitoring_cadvisor`, and `monitoring_node_exporter`
- End-to-end media stack details live in [[personal/tech/media-stack|Media Stack]]

## Machine Inventory

### proxmox.markul.net

- Type: Physical machine
- Role: Proxmox host
- Network: `192.168.222.3`
- Compute: `Intel Xeon E5-2666 v3`, `20` threads / `10` cores / `1` socket, `31 GiB` RAM
- Storage: `~1 TB NVMe` (`local` root + `local-lvm` thin pool), `~500 GB` ext4 disk mounted as `hdd500`, `~1 TB` LVM-thin pool `hdd1000`, and `~2 TB` LVM disk `usb-drive2000`
- Hosted workloads: VMs `100`, `101`, `102`
- Summary: dedicated hypervisor node, not an app-container host

### dev.markul.net

- Type: VM `100` on `proxmox.markul.net`
- Network: LAN `192.168.222.8`, public SSH forward `46.191.235.87:55755`
- Compute: `Intel Xeon E5-2666 v3` virtual CPU, `6` vCPU, `10 GiB` RAM
- Storage: `250 GB` virtual disk
- Summary: main application and integration host with the largest Docker footprint

### raspberrypi.markul.net

- Type: Physical single-board computer
- Network: LAN `192.168.222.40`, SSH as `pi@raspberrypi.markul.net`
- Compute: `Raspberry Pi 4 Model B Rev 1.2`, `arm64`, `4` cores, `3.7 GiB` RAM
- Storage: `116.1 GB` SD card with `512 MB` boot partition and `115.6 GB` root partition
- Known note evidence:
  - Included in the always-on baseline described in [[personal/tech/electricity-consumption|electricity-consumption]]
- Summary: lightweight always-on ARM host for small app, media, monitoring, and utility containers

### alfa.markul.net

- Type: VM `101` on `proxmox.markul.net`
- Network: LAN `192.168.222.16`, public SSH forward `46.191.235.87:55955` intended / unverified
- Compute: `QEMU Virtual CPU`, `10` vCPU, guest-visible `~7.6 GiB` RAM, configured `16192 MiB` max with ballooning
- Storage: `200 GB` virtual disk
- Summary: active Kubernetes dev/test VM

### infra.markul.net

- Type: VM `102` on `proxmox.markul.net`
- Network: LAN `192.168.222.5`
- Compute: `QEMU Virtual CPU`, `2` vCPU, `~3.8 GiB` RAM
- Storage: `940 GB` virtual disk + `1.8 TB` mounted data disk
- Live compose root on host: `/home/marat/docker`
- Summary: media/download and utility stack

### optiplex.markul.net

- Type: Physical machine
- Network: LAN `192.168.222.2`, public SSH forward `46.191.235.87:55855`
- Compute: `Intel Core i5-13500T`, `20` threads / `14` cores / `1` socket, `62 GiB` RAM
- Storage: `~1 TB NVMe` + `~1 TB` SSD
- Summary: newer multi-purpose host for Docker, local Kubernetes, media, and CI

### frankfurt.markul.net

- Type: VPS
- Network: hostname `739712.cloud4box.ru`
- Compute: `QEMU Virtual CPU`, `1` core / `1` thread / `1` socket, `x86_64`, `957 MiB` RAM
- Storage: `20 GB` system disk
- Summary: small public VPN / edge utility node

### helsinki.markul.net

- Type: VPS
- Network: public IP `65.108.89.190`, hostname `helsinki-arm`
- Compute: `Neoverse-N1`, `4` cores / `4` threads / `1` socket, `aarch64`, `7.5 GiB` RAM
- Storage: `~76 GB` system disk
- Summary: public ARM edge node for reverse proxying, TLS, CI, VPN, and local failed-request log collection
