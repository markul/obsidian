# 2026-04-02 my-infra-bootstrap

## Goal

- Create a stable agent project for personal infrastructure work and capture the current durable context

## Scope

- Agent project: [[agents/projects/my-infra|My Infra]]
- Repo: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[personal/tech/Infrastructure|Infrastructure]]
  - [[personal/tech/hardware|hardware]]
  - [[personal/tech/electricity-consumption|electricity-consumption]]

## Actions

- Checked the existing `agents/projects/` area and confirmed there was no existing `my-infra` project
- Read the current infrastructure, hardware, and electricity notes
- Created the initial `My Infra` agent project workspace, later consolidated into [[agents/projects/my-infra|My Infra]]
- Captured the current machine inventory and note-based scope in the new project
- Recorded the current tariff mismatch in the project as an open note-quality issue
- Added a link from [[personal/tech/Infrastructure|Infrastructure]] to the new agent project
- Added a minimal daily-note entry for the new project

## Decisions

- Use a note-based agent project for personal infra tracking instead of assuming a separate code repository
- Keep [[personal/tech/Infrastructure|Infrastructure]] as the canonical machine inventory and use the project for resumable cross-note context

## Follow Up

- Recalculate [[personal/tech/electricity-consumption|electricity-consumption]] using the current `3.5 RUB/kWh` tariff
- Decide whether `Infrastructure.md` should stay as a flat machine list or be split into network, services, and capacity sections later
