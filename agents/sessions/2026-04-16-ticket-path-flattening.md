---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-16
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---

# 2026-04-16 ticket path flattening

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[agents/projects/second-brain|Second Brain]]
  - [[work/alfa-bank/index|Alfa Bank]]
  - [[daily/2026-04-16]]

## Goal

- Flatten Alfa ticket note paths to one stable `tickets/` directory and keep status grouping in the index plus frontmatter instead of folder names

## Actions

- Replaced the vault ticket-path convention in [[AGENTS]] so ticket notes now live directly under `work/alfa-bank/tickets/`
- Updated the Alfa ticket index to group notes by `status` while linking to stable ticket paths
- Updated the ticket template to preserve the stable-path convention for future notes
- Retargeted existing wiki-links that encoded ticket status in the path
- Moved the existing Alfa ticket notes out of `tickets/done/` and `tickets/on-hold/` into the stable `tickets/` directory
- Removed the now-unused status subdirectories after the ticket files were moved

## Findings

- Path-encoded ticket status created avoidable churn in links, daily notes, and session notes whenever local status changed
- The vault already keeps ticket `status` in frontmatter, so folder-based status duplicated the same fact in a less stable place
- A stable ticket directory plus status-grouped index matches the broader vault preference for stable placement with links and metadata carrying meaning

## Decisions

- Keep Alfa ticket notes in one stable directory: `work/alfa-bank/tickets/`
- Use ticket frontmatter `status` and the Alfa index as the canonical local grouping mechanism
- Do not move ticket files on future local status changes unless there is a separate archival need

## Validation

- `rg -n 'work/alfa-bank/tickets/(new|active|on-hold|done)/' /home/marat/dev/git/markul/obsidian -g '*.md'`
- `find /home/marat/dev/git/markul/obsidian/work/alfa-bank/tickets -maxdepth 2 -type f | sort`
