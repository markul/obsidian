---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-03-31
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---
# 2026-03-31 obsidian-vault-and-work-planning

## Goal

- Reorganize the Obsidian vault structure
- Set up and document one-way vault sync from `optiplex.markul.net` to the MacBook as personal vault maintenance
- Separately capture work context and handling rules for Jira and Confluence

## Scope

- Repo: this Obsidian vault
- Related notes:
  - [[daily/2026-03-31]]
  - [[agents/projects/second-brain|Second Brain]]

## Actions

- Reworked the vault layout around `agents/`, `daily/`, `personal/tech/`, and `work/`
- Renamed files and directories to avoid spaces and updated wiki links
- Added daily note structure and recorded the current day in [[daily/2026-03-31]]
- Captured vault sync setup as personal vault work now owned by [[agents/projects/second-brain|Second Brain]]
- Verified `snxctl status` before work-related actions
- Recorded that Jira and Confluence items are work-related and should use `utk-mcp`
- Captured today’s work intention: finish `UTK2-3264` and switch to `UTK2-3243`
- Structured work tickets into per-ticket directories with `description.md`, `plan.md`, and `result.md`

## Decisions

- Agent projects live under stable notes in `agents/projects/`
- File and directory names should use kebab-case without spaces
- Vault sync is a personal vault workflow, not a work item
- Vault sync is one-way from `optiplex.markul.net` to the MacBook
- Sync excludes `.git/` and `.obsidian/`
- Sync runs on demand from Obsidian using `Shell Commands` and `Commander`
- Before work-related requests, verify `snxctl` and keep Jira/Confluence under `work/` context

## Follow Up

- Finish work on `UTK2-3264`
- Switch to `UTK2-3243`
- Use `utk-mcp` for Jira and Confluence requests
