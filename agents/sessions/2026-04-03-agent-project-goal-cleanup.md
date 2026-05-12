---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-03
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---
# Agent project goal cleanup

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`

## Goal

- Move the generic session-resumability requirement out of individual project goals and into the vault instructions

## Actions

- Updated `AGENTS.md` to state that agent projects preserve resumable context across sessions by default
- Updated the `obsidian-vault-workflow` skill to match the same rule
- Trimmed current project notes so their `Goal` sections describe the actual project objective instead of vault-maintenance behavior

## Result

- Agent project goals now describe only the real project objective
- The resumability requirement is centralized in the vault instructions instead of being repeated in each project
