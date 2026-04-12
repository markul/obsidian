# 2026-04-02 my-infra-public-security-review

## Goal

- Review the publicly accessible infrastructure footprint and identify obvious security issues without making destructive changes

## Scope

- Agent project: [[agents/projects/my-infra|My Infra]]
- Repo: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[personal/tech/Infrastructure|Infrastructure]]
  - [[agents/sessions/2026-04-02-my-infra-inventory-cleanup|2026-04-02-my-infra-inventory-cleanup]]

## Actions

- Enumerated reachable public TCP ports for:
  - `46.191.235.87`
  - `739712.cloud4box.ru`
  - `65.108.89.190`
- Confirmed external exposure:
  - `46.191.235.87` -> `22`, `80`, `443`, `55755`, `55855`, `55955`
  - `739712.cloud4box.ru` -> `22`
  - `65.108.89.190` -> `22`, `80`, `443`
- Collected SSH banners:
  - `46.191.235.87:22` -> `SSH-2.0-server`
  - `46.191.235.87:55755` -> `OpenSSH_9.6p1 Ubuntu-3ubuntu13.15`
  - `46.191.235.87:55855` -> `OpenSSH_8.9p1 Ubuntu-3ubuntu0.14`
  - `46.191.235.87:55955` -> `OpenSSH_9.6p1 Ubuntu-3ubuntu13.15`
  - `739712.cloud4box.ru:22` -> `OpenSSH_8.9p1 Ubuntu-3ubuntu0.14`
  - `65.108.89.190:22` -> `OpenSSH_9.6p1 Ubuntu-3ubuntu13.15`
- Compared SSH host keys and confirmed:
  - `46.191.235.87:55755` matches `dev`
  - `46.191.235.87:55855` matches `optiplex`
  - `46.191.235.87:55955` matches `alfa`
  - `739712.cloud4box.ru:22` matches `frankfurt.markul.net`
  - `65.108.89.190:22` matches `helsinki.markul.net`
- Probed advertised public SSH auth methods with non-destructive auth negotiation:
  - `46.191.235.87:22` -> `publickey,password`
  - `739712.cloud4box.ru:22` -> `publickey,password`
  - `65.108.89.190:22` -> `publickey`
  - `46.191.235.87:55755/55855/55955` -> `publickey`
- Checked HTTP responses:
  - `http://46.191.235.87` -> `503` from `nginx`
  - `https://46.191.235.87` -> `503` from `nginx`
  - `http://65.108.89.190` -> `503` from `nginx`
  - `https://65.108.89.190` requires SNI and does not serve a useful response by bare IP
- SSHed into `dev` and `alfa` through trusted paths and checked listeners and basic hardening state
- Observed on both `dev` and `alfa`:
  - `ufw` enabled and active
  - `unattended-upgrades` enabled and active
- Observed on `dev`:
  - Many services listen on `0.0.0.0` internally, including `1433`, `5432`, `9200`, `9090`, `9092`, `9093`, `3000`, `8000`, `8033`, `8081`, and `8090`
  - Docker publishes these services broadly on the host, but the external scan did not show them on the internet-facing IP
- Observed on `alfa`:
  - Only `22/tcp` was listening
- Attempted authenticated SSH access from this environment:
  - `dev` and `alfa` succeeded
  - `optiplex` public forward rejected the current key with `publickey` only
  - `frankfurt` rejected login and advertised `publickey,password`
  - `helsinki` rejected login and advertised `publickey` only
- This workstation currently reports SSH host-key changes for:
  - `frankfurt.markul.net`
  - `helsinki.markul.net`

## Decisions

- Treat the undocumented public SSH service on `46.191.235.87:22` as a real security finding until it is identified and justified
- Treat public `password` authentication on `46.191.235.87:22` and `739712.cloud4box.ru:22` as a hardening gap unless there is a specific operational need
- Treat the host-key mismatch warnings for `frankfurt` and `helsinki` as a trust problem that must be resolved before normal SSH use resumes from this workstation

## Follow Up

- Identify what serves `46.191.235.87:22` and decide whether to close it or force `publickey` only
- Disable public SSH password auth on `739712.cloud4box.ru` if it is not intentionally required
- Verify and re-trust the current SSH host keys for `frankfurt.markul.net` and `helsinki.markul.net` only after confirming they were legitimately reprovisioned or rekeyed
- Review whether the broad `0.0.0.0` binds on `dev` should be tightened to loopback or Docker-internal networking where possible
