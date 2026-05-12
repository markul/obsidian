---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-03
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---
# Agent project instructions update

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`

## Goal

- Align the vault rules and the `obsidian-vault-workflow` skill with a single-file agent project model

## Actions

- Updated `AGENTS.md` to replace the old `agents/projects/{name}/` three-file model with single-file project notes at `agents/projects/{name}.md`
- Added the fixed section order for agent project notes: `Status`, `Timestamps`, `Goal`, `Scope`, `Current State`, `Key Decisions`, `Plan`, `Blockers`
- Clarified that `Plan` should be a short numbered list
- Clarified that `Blockers` should contain only real current blockers and should be cleaned up when the blocker is gone
- Updated the `obsidian-vault-workflow` skill to match the same model

## Decision

- Agents should treat agent projects as single durable notes rather than three-file workspaces
