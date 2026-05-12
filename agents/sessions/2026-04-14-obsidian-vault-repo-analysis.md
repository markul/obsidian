---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-14
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---

# 2026-04-14 obsidian-vault-repo-analysis

## Goal

- Analyze the vault repository structure, current note organization, and obvious consistency issues.

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[AGENTS]]
  - [[daily/2026-04-14]]
  - [[work/alfa-bank/index|Alfa Bank]]

## Actions

- Read the vault `AGENTS.md` and the `obsidian-vault-workflow` skill guidance.
- Reviewed the workflow checklist and compared it with the current vault policy.
- Inspected the top-level folder layout and markdown file inventory.
- Sampled representative agent project, session, ticket, template, and daily notes.
- Checked `git status --short` to understand active note churn and local moves.
- Verified that the vault currently has 93 markdown files, with most activity concentrated in `agents/sessions/`, `agents/projects/`, and Alfa work tickets.
- Identified policy drift where the skill checklist and `agents/templates/project-template.md` still describe the older multi-file `project.md` layout, while the vault policy now uses single-file project notes under `agents/projects/{name}.md`.
- Identified stale historical references to `description.md`, `plan.md`, and `result.md` in the workflow checklist and older session notes.
- Identified two new session notes that use `note-type: session` instead of `note-type: agent-session`.
- Identified that [[work/alfa-bank/index|Alfa Bank]] still lists `UTK2-3290` and `UTK2-3295` as active even though local ticket notes have been moved to `done`.
- Verified that [[work/alfa-bank/tickets/utk2-3290|UTK2-3290]] and [[work/alfa-bank/tickets/utk2-3295|UTK2-3295]] currently fail YAML frontmatter parsing because the `status` key is indented with a leading space.
- Fixed the invalid frontmatter in [[work/alfa-bank/tickets/utk2-3290|UTK2-3290]] and [[work/alfa-bank/tickets/utk2-3295|UTK2-3295]].
- Updated [[work/alfa-bank/index|Alfa Bank]] so the active and done ticket lists match the current ticket locations.
- Normalized the two `2026-04-13` session notes to `note-type: agent-session` and added the standard session frontmatter fields.
- Updated [[agents/templates/project-template|project template]] to match the single-file `agents/projects/{name}.md` project model.
- Updated the external `obsidian-vault-workflow` checklist to match the vault's single-file project and ticket model.
- Added a short `Context Budget` section to [[AGENTS]] to define when notes should be updated and how to keep note content compact.

## Decisions

- Treat `AGENTS.md` as the canonical vault policy when checklist or template guidance disagrees.
- Treat the current vault model as stable: single-file agent projects, single-file work tickets, lightweight daily notes, and detailed session notes.
- Treat note metadata naming as part of the schema; `agent-session` should remain the canonical session `note-type`.

## Follow Up

- None.
