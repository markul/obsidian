---
tags:
  - agents/project
  - status/active
---

# OpenCode Multi-Agent

## Status

- `active`

## Timestamps

- Created: `2026-04-03 19:00`
- Completed: `not completed`

## Goal

- Learn how to set up and use role-based multi-agent workflows in OpenCode

## Plan

1. Add the practice-repo orchestrator primary and decide how closely it should mirror the global `architect` agent.
2. Validate manual `@` invocation and automatic task delegation across the global and repo-local agent sets.
3. Capture the working role boundaries and setup patterns in `docs/agents.md`.

## Scope

- Practice repo: `/home/marat/dev/git/markul/opencode`
- Learning note: [[personal/learning/ai/multi-agent|multi-agent]]
- Related notes:
  - [[daily/2026-04-03]]

## Validation

- `cd /home/marat/dev/git/markul/opencode && opencode`

## Current State

- Practice repo initialized at `/home/marat/dev/git/markul/opencode` on `master`
- `opencode.json` loads `docs/instructions.md` via the `instructions` field
- Shared personal agent guidance now has a documented single point of truth under `~/.agents/`
- Global OpenCode config now exists at `~/.config/opencode/opencode.json`, loads `/home/marat/.agents/AGENTS.md`, and sets `architect` as the default agent
- Built-in OpenCode agents `build`, `compaction`, `explore`, `general`, `plan`, `summary`, and `title` are disabled in the global config so the personal agent set is the visible default
- Global primary agents now exist under `~/.config/opencode/agents/`: `architect`, `advisor`, `chat`
- Global subagents now exist under `~/.config/opencode/agents/`: `coder-glm5`, `coder-codex`, `coder-sonnet45`, `reviewer-opus46`
- Codex still keeps runtime-local settings in `~/.codex/config.toml`, and `~/.codex/AGENTS.md` is symlinked to `~/.agents/AGENTS.md` so both runtimes share the same base guidance
- `architect` uses `github-copilot/gpt-5.4` as the orchestrator that can delegate to the coder and reviewer subagents via `permission.task`
- `advisor` uses `github-copilot/claude-sonnet-4.5` as the read-only planning and strategy primary
- `chat` uses `github-copilot/claude-sonnet-4.6` as the general conversation primary
- `coder-glm5` uses `alfagen/zai-org/GLM-5-FP8`; `coder-codex`, `coder-sonnet45`, and `reviewer-opus46` use the expected GitHub Copilot models for implementation and review
- `docs/agents.md` contains the full reference: agent types, config options, role patterns
- `.opencode/agents/reviewer.md` — read-only code review subagent defined
- `.opencode/agents/planner.md` — planning subagent with restricted bash (git/grep/find) defined
- Orchestrator agent not yet added to the practice repo, so the next major implementation step remains repo-local orchestration and delegation validation

## Key Decisions

- Keep reference documentation in `docs/agents.md` inside the practice repo rather than in the Obsidian note
- Treat `~/.agents/` as the single point of truth for shared personal agent guidance across runtimes
- Use a symlink from `~/.codex/AGENTS.md` to `~/.agents/AGENTS.md` instead of maintaining a separate Codex copy
- Use `.opencode/agents/` markdown format for agent definitions (not `opencode.json` agent blocks)
- Use global markdown agent files in `~/.config/opencode/agents/` for the reusable personal agent set
- Disable the built-in OpenCode agents in the global config so the custom personal agent set stays visible by default
- Keep Codex-specific runtime configuration in `~/.codex/` and OpenCode-specific runtime wiring in `~/.config/opencode/`, rather than forcing full file-level unification
- Use `advisor` instead of `planner` for the second custom primary to avoid confusion with OpenCode's built-in `plan` agent
- Use `reviewer-opus46` for the default global reviewer because `github-copilot/claude-opus-4.6` is the strongest review model currently available in this OpenCode setup
- Use `github-copilot/claude-sonnet-4.6` for the general `chat` primary because it is a better balance of quality and latency than `claude-opus-4.6` for everyday conversation

## Blockers

- None currently
