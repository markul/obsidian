---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-03
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---
# Agent project single-file migration

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`

## Goal

- Migrate existing agent projects from the old three-file directory layout to the current single-file note model

## Actions

- Consolidated the current agent projects into single notes at `agents/projects/{name}.md`
- Rewrote vault links that still pointed to `agents/projects/{name}/project`
- Removed the obsolete `project.md`, `plan.md`, and `result.md` files and their empty directories

## Result

- The current project set now follows the same single-file model already defined in `AGENTS.md` and the `obsidian-vault-workflow` skill
