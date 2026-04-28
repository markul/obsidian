# Software

## Related Notes

- [[personal/tech/index|Tech]]
- [[personal/tech/Infrastructure|Infrastructure]]
- [[personal/learning/ai/index|AI]]

## Purpose

- This note tracks host-level software inventory, runtimes, and notable installed or running platform software
- Network topology, hardware, and machine roles live in [[personal/tech/Infrastructure|Infrastructure]]

## proxmox.markul.net

- Platform software:
  - `Proxmox VE`
  - `node_exporter` as a systemd service on port `9100`
- Runtime state:
  - no Docker or Podman containers detected
  - no LXC containers detected

## dev.markul.net

- Runtime:
  - Docker
  - no Kubernetes workloads detected
- Main application stack:
  - `markul.engine`
  - `markul.api`
  - `markul.media`
  - `markul.web`
  - `markul.canary`
  - `markul.instagram`
  - `identity`
  - `identity-api`
  - `sts`
- Data and messaging:
  - `postgres`
  - `pgadmin`
  - `consul`
  - `kafka`
  - `kafka-ui`
  - `redpanda`
- Observability:
  - `grafana`
  - `grafana_renderer`
  - `monitoring_prometheus`
  - `monitoring_cadvisor`
  - `monitoring_node_exporter`
  - `kibana`
  - `elasticsearch`
  - `loki-dev`
  - `alloy-logs`
- Edge and forwarding:
  - `nginx-proxy`
  - `nginx-proxy-letsencrypt`
  - `watchtower`
  - `portainer`
  - `socat-jellyseer`
  - `socat-jellyfin`
  - `socat-registry`
  - `socat-drone-proxy`
  - `socat-vagrant`
- CI and misc:
  - `drone-runner`
  - `ut99-server`
- Logging details:
  - local `Loki + Alloy` tails the dedicated `nginx-proxy` failed-only access log
  - Loki is exposed only on `127.0.0.1:3101` and Docker network `loki-dev:3100`
  - shared `Failed Requests` dashboard is `https://grafana.markul.net/d/failed-requests/failed-requests`

## raspberrypi.markul.net

- Runtime:
  - Docker
  - no Podman or Kubernetes tooling detected
- Containers:
  - `watchtower`
  - `portainer_edge_agent`
  - `mediamtx`
  - `markul.client.dev`
  - `markul.auth.proxy.dev`
  - `monitoring_cadvisor`
  - `monitoring_node_exporter`

## alfa.markul.net

- Runtime:
  - `minikube`
- Kubernetes workloads:
  - `ufr-kpnsb-limit-service` in `default`
  - standard system pods in `kube-system`

## infra.markul.net

- Runtime:
  - Docker
- Live compose root:
  - `/home/marat/docker`
- Active compose projects:
  - inferred from running containers; exact compose project names not revalidated here
- Notable services:
  - `sonarr`
  - `radarr`
  - `prowlarr`
  - `qbittorrent`
  - `wg-easy`
  - `proxy-tinyproxy-1`
  - `registry`
  - `portainer_edge_agent`
  - `mtproto-proxy`
  - `nginx-proxy`
  - `ftp2`

## optiplex.markul.net

- Main purpose: `.NET` development and learning AI agents
- Editor: Visual Studio Code
- VS Code extensions:
  - Codex
  - Copilot Chat
  - Kilo Code
- Container and local cluster tooling:
  - Docker
  - Minikube
- AI tooling:
  - Standalone Codex app
  - OpenCode
- Runtime:
  - Docker
  - `minikube`
- Containers:
  - `drone-runner`
  - `vagrant-boxes`
  - `monitoring_cadvisor`
  - `monitoring_node_exporter`
  - `portainer_edge_agent`
  - `jellyfin`
  - `seerr`
  - `minikube`
  - `utk-mcp-utk-mcp-1`
- Kubernetes workloads:
  - `skp-product-change-workflow-service-86d49bf4bb-t5x7d` in `default`
  - `coredns`
  - `etcd`
  - `kube-apiserver`
  - `kube-controller-manager`
  - `kube-proxy`
  - `kube-scheduler`
  - `storage-provisioner`

## frankfurt.markul.net

- Runtime:
  - Docker
  - `node_exporter` as a systemd service on port `9100`
- Containers:
  - `portainer_edge_agent`
  - `amnezia-awg`

## helsinki.markul.net

- Runtime:
  - Docker
  - `node_exporter` as a systemd service on port `9100`
- Containers:
  - `portainer_edge_agent`
  - `wireguard`
  - `drone-runner`
  - `drone-server`
  - `nginx-proxy-letsencrypt`
  - `nginx-proxy`
  - `registry-arm`
  - `loki`
  - `alloy-logs`
  - `ftp`
- Observability details:
  - local `Loki + Alloy` tails the dedicated `nginx-proxy` failed-only access log
  - Loki is exposed on port `3100` for `dev.markul.net` Grafana only
  - failed-request log path is `/home/marat/dev/git/docker/docker-compose/nginx-proxy/logs/nginx-proxy/failed-requests.log`
- Registry details:
  - `/home/marat/dev/git/docker/docker-compose/hub-arm` runs a `registry:2` service behind `nginx-proxy`
  - public endpoint is `https://hub-arm.markul.net/v2/`
  - host port `5000` is intentionally not published
  - `Markul.Arm` now publishes `arm64` images here, including `:dev` plus versioned build tags; `amd64` images remain on `hub.markul.net`
