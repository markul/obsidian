---
tags:
  - agents/project
note-type: agent-project
status: active
created: 2026-04-02T19:22
completed: 
related-ticket: 
---

# My Infra

## Status

- `active`

## Timestamps

- Created: `2026-04-02 19:22`
- Completed: `not completed`

## Goal

- Track the home and VPS infrastructure under `markul.net`

## Plan

1. Reconcile [[personal/tech/electricity-consumption|electricity-consumption]] with the current tariff and saved measurements.
2. Continue tightening the public SSH exposure and trust state for the externally reachable hosts.
3. Capture future consolidation or migration decisions here instead of scattering them across session notes.

## Scope

- Hub note: [[personal/tech/index|Tech]]
- Area note: [[personal/tech/Infrastructure|Infrastructure]]
- Related notes:
  - [[personal/tech/hardware|hardware]]
  - [[personal/tech/electricity-consumption|electricity-consumption]]
  - [[daily/2026-04-02]]
  - [[daily/2026-04-03]]

## Validation

## Current State

- This project is note-based inside the Obsidian vault; there is no separate `my-infra` repo tracked here
- [[personal/tech/Infrastructure|Infrastructure]] is organized into `Overview`, `Network`, `Service Placement`, and `Machine Inventory`
- Current confirmed machines include `proxmox.markul.net`, `raspberrypi.markul.net`, `dev.markul.net`, `alfa.markul.net`, `infra.markul.net`, `optiplex.markul.net`, `frankfurt.markul.net`, and `helsinki.markul.net`
- `raspberrypi.markul.net` is now inventoried from live SSH data
- [[personal/tech/electricity-consumption|electricity-consumption]] still needs reconciliation with the current `3.5 RUB/kWh` tariff

## Key Decisions

- Keep personal infrastructure tracking in the Obsidian vault rather than creating a separate repo-specific project
- Treat [[personal/tech/Infrastructure|Infrastructure]] as the primary machine inventory note

## Blockers

- None currently
