---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-02
service: 
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# 2026-04-02 my-infra-inventory-cleanup

## Goal

- Turn `infrastructure.md` into a cleaner infrastructure inventory with clearer network and service placement sections

## Scope

- Agent project: [[agents/projects/my-infra|My Infra]]
- Repo: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[personal/tech/infrastructure|Infrastructure]]
  - [[personal/tech/hardware|hardware]]
  - [[personal/tech/electricity-consumption|electricity-consumption]]

## Actions

- Read the existing flat machine list in [[personal/tech/infrastructure|Infrastructure]]
- Reorganized the note into `Overview`, `Network`, `Service Placement`, and `Machine Inventory`
- Pulled public endpoints and LAN addresses into dedicated network sections
- Grouped workloads by function instead of leaving everything only under per-host summaries
- Kept the per-machine details, but reduced repeated phrasing and made the runtime role of each host easier to scan
- SSHed into `pi@raspberrypi.markul.net` and captured hardware, network, storage, Docker, and running-service details
- Updated `raspberrypi.markul.net` with LAN IP `192.168.222.40`, `Raspberry Pi 4 Model B Rev 1.2`, `arm64`, `4` cores, `3.7 GiB` RAM, `116.1 GB` SD storage, and the current Docker container set
- Updated the `my-infra` project note to record that the inventory cleanup is done

## Decisions

- Keep [[personal/tech/infrastructure|Infrastructure]] as the canonical inventory note instead of splitting it into multiple machine notes
- Use service placement sections for the high-level operational view and machine sections for the detailed facts

## Follow Up

- Recalculate [[personal/tech/electricity-consumption|electricity-consumption]] using the current `3.5 RUB/kWh` tariff
- Decide whether to add a dedicated storage/capacity section later if disk-planning becomes an active task
- If useful later, split Raspberry Pi services into app-facing versus utility-only groups in the inventory
