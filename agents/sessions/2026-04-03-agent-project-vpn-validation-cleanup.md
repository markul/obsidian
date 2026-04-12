---
tags:
  - agents/session
---

# Agent Project VPN Validation Cleanup

## Scope

- Alfa-work guidance and the `UTK2-3264` agent project note

## Goal

- Keep VPN checking as shared Alfa-work guidance instead of duplicating it inside project-specific `Validation`

## Actions

- Updated [AGENTS.md](/home/marat/dev/git/markul/obsidian/AGENTS.md) to state that the VPN check belongs in shared Alfa-work guidance
- Updated the matching guidance in `/home/marat/.codex/skills/obsidian-vault-workflow/SKILL.md`
- Removed `snxctl status` from the `Validation` section in [utk2-3264.md](/home/marat/dev/git/markul/obsidian/agents/projects/utk2-3264.md)
- Recorded the cleanup in today's daily note

## Result

- Common Alfa VPN preconditions now live only in the shared guidance, while project `Validation` stays task-specific
