---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-11
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---

# 2026-04-11 ticket done project done

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[agents/projects/utk2-3264|UTK2-3264 Agent Work]]
  - [[work/alfa-bank/tickets/utk2-3264|UTK2-3264]]

## Goal

- Align the vault workflow so agent projects move to `done` when the corresponding local ticket note moves to `done`

## Actions

- Updated `AGENTS.md` to make the ticket-to-project `done` transition explicit in both the agent-project and work-ticket rules
- Reviewed the current done-ticket set against `agents/projects/` to find any mismatches
- Marked [[agents/projects/utk2-3264|UTK2-3264 Agent Work]] as `done`
- Set the project `Completed` timestamp and updated its ticket link to the `tickets/done` path

## Findings

- `UTK2-3264` was the only current mismatch: its ticket note was already under `work/alfa-bank/tickets/`, but the agent project still used `status/active`
- `UTK2-3274` was already aligned as `done`

## Decisions

- Treat a local ticket move to `done` as requiring the linked agent project to be closed in the same edit pass
