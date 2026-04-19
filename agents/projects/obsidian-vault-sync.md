---
tags:
  - agents/project
note-type: agent-project
status: done
created: 2026-03-31T00:00
completed: 2026-03-31T00:00
related-ticket: 
---

# Obsidian Vault Sync

## Status

- `done`

## Timestamps

- Created: `2026-03-31 00:00`
- Completed: `2026-03-31 00:00`

## Goal

- Keep the vault on the MacBook synchronized from this machine
- Support both manual and recurring sync from inside Obsidian on the MacBook

## Plan

1. Revisit this only if the sync command, excludes, interval, or Obsidian integration need to change.

## Scope

- Sync source: `/home/marat/dev/git/markul/obsidian`
- Source host: `optiplex.markul.net`
- Destination path on the MacBook: `/Users/marat/dev/git/markul/obsidian/`
- Related notes:
  - [[daily/2026-03-31]]
  - [[agents/sessions/2026-03-31-obsidian-vault-and-work-planning]]

## Validation

## Current State

- One-way sync is implemented with `rsync` over SSH
- Sync excludes `.git/` and `.obsidian/`

## Key Decisions

- Keep sync one-way only: the MacBook pulls from this machine
- Use plain `rsync` over SSH rather than adding a heavier sync layer
- Use the same command for both manual and recurring sync triggers

## Blockers

- None
