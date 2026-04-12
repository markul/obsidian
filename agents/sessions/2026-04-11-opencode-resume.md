---
tags:
  - agents/session
---

# 2026-04-11 opencode resume

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`
- Practice repo: `/home/marat/dev/git/markul/opencode`
- Related notes:
  - [[agents/projects/opencode|OpenCode Multi-Agent]]
  - [[daily/2026-04-11]]

## Goal

- Resume the OpenCode multi-agent practice project from the existing durable project note

## Actions

- Reviewed [[agents/projects/opencode|OpenCode Multi-Agent]] to recover the last durable project state
- Checked the practice repo status to confirm whether the local implementation had moved since the project note was written
- Re-read `opencode.json`, `docs/instructions.md`, and the current `.opencode/agents/` definitions to verify the configured workflow pieces
- Confirmed that a machine-level OpenCode config was added at `~/.config/opencode/opencode.json`
- Set `architect` as the default global OpenCode agent in `~/.config/opencode/opencode.json`
- Disabled the built-in OpenCode agents `build`, `plan`, `general`, `explore`, `summary`, `title`, and `compaction` in the global config
- Created a reusable global agent set under `~/.config/opencode/agents/`
- Added primary agents `architect` and `advisor`
- Added a general `chat` primary using `github-copilot/claude-sonnet-4.6`
- Added coder subagents `coder-glm5`, `coder-codex`, and `coder-sonnet45`
- Added a global `reviewer-opus46` subagent using `github-copilot/claude-opus-4.6`
- Synced the durable vault notes so the project note and learning note now reflect the live global OpenCode setup
- Added a documented single-point-of-truth section for shared agent guidance plus Codex/OpenCode-specific tweak notes
- Noted that Codex uses a symlink from `~/.codex/AGENTS.md` to `~/.agents/AGENTS.md`

## Findings

- The practice repo is still on `master` with no commits and untracked project files
- `opencode.json` still loads `docs/instructions.md` through the `instructions` field
- A global config file now exists at `~/.config/opencode/opencode.json`
- The global config now sets `architect` as the default agent
- The global config disables the visible built-in OpenCode agents so only the custom personal agent set remains available
- OpenCode global agents are loaded from `~/.config/opencode/agents/{name}.md`
- Shared personal agent guidance is best treated as living under `~/.agents/`, while runtime-specific wiring stays under `~/.codex/` and `~/.config/opencode/`
- Codex shares the same base AGENTS instructions by symlinking `~/.codex/AGENTS.md` to `~/.agents/AGENTS.md`
- The available provider set supports all requested coder models through `alfagen` and `github-copilot`
- `github-copilot/claude-sonnet-4.6` is the best current fit for a broad conversation primary; `claude-opus-4.6` would be the heavier but slower alternative
- `.opencode/agents/reviewer.md` and `.opencode/agents/planner.md` are present and match the project note
- The orchestrator primary agent has not been added yet, so project step 3 remains the next major implementation task
- The durable notes now capture the global default agent, disabled built-ins, and the current primary/subagent lineup
- The Codex AGENTS drift issue is handled by symlinking `~/.codex/AGENTS.md` to the shared `~/.agents/AGENTS.md`

## Decisions

- Treat [[agents/projects/opencode|OpenCode Multi-Agent]] as the active personal project context for the current session
- Use `advisor` instead of `planner` for the second custom primary to avoid confusion with OpenCode's built-in `plan` agent
- Document the desired cross-runtime shape as shared guidance in `~/.agents/` plus thin runtime-specific layers for Codex and OpenCode
