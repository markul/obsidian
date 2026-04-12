---
tags:
  - agents/session
---

# Agent Project Validation Minimal Commands Update

## Scope

- Agent project validation guidance and the `UTK2-3264` project note

## Goal

- Define `Validation` as the minimal required verification commands and trim `UTK2-3264` to the actual two-command flow

## Actions

- Updated [AGENTS.md](/home/marat/dev/git/markul/obsidian/AGENTS.md) to define `Validation` as minimal required verification commands
- Updated the matching guidance in `/home/marat/.codex/skills/obsidian-vault-workflow/SKILL.md`
- Reduced the `Validation` section in [utk2-3264.md](/home/marat/dev/git/markul/obsidian/agents/projects/utk2-3264.md) to:
  - `skaffold run -p api --port-forward`
  - `dotnet test test/Ufr.Kpnsb.LimitService.IntegrationTests`
- Recorded the refinement in today's daily note

## Result

- Project validation guidance is now narrower and `UTK2-3264` reflects only the essential verification commands
