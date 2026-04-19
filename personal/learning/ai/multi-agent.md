# Multi-Agent OpenCode

Reference: `/home/marat/dev/git/markul/opencode/docs/agents.md`
Agent project: [[agents/projects/opencode|OpenCode Multi-Agent]]

## Related Notes

- [[personal/learning/ai/index|AI]]
- [[personal/learning/ai/second-brain|Second Brain]]
- [[agents/projects/opencode|OpenCode Multi-Agent]]
- [[personal/tech/software|Software]]

## Global Agent Config — Single Point of Truth

- Shared personal agent guidance should live under `~/.agents/`
- Canonical shared files:
  - `~/.agents/AGENTS.md` — shared base instructions
  - `~/.agents/skills/{name}/SKILL.md` — shared custom skills
- Keep runtime-specific files limited to wiring, permissions, providers, and other tool-only behavior

### Codex Tweaks Required

- Symlink `~/.codex/AGENTS.md` to `~/.agents/AGENTS.md` so Codex loads the shared base guidance from the same single source of truth

### OpenCode Tweaks Required

- In `~/.config/opencode/opencode.json`, added `/home/marat/.agents/AGENTS.md` to the `instructions` array so every OpenCode session loads the shared base guidance from `~/.agents/`

## Current Personal Setup

- Global config: `~/.config/opencode/opencode.json`
- Shared instructions: `/home/marat/.agents/AGENTS.md`
- Default global agent: `architect`
- Built-in OpenCode agents disabled in the global config: `build`, `compaction`, `explore`, `general`, `plan`, `summary`, `title`

## Global Agents

- Primary agents:
  - `architect` — `github-copilot/gpt-5.4`; orchestrator that can delegate to `coder-glm5`, `coder-codex`, `coder-sonnet45`, and `reviewer-opus46`
  - `advisor` — `github-copilot/claude-sonnet-4.5`; read-only planning and strategy primary
  - `chat` — `github-copilot/claude-sonnet-4.6`; general conversation primary
- Subagents:
  - `coder-glm5` — `alfagen/zai-org/GLM-5-FP8`; fast drafting and quick iterations
  - `coder-codex` — `github-copilot/gpt-5.3-codex`; strong implementation and debugging
  - `coder-sonnet45` — `github-copilot/claude-sonnet-4.5`; careful maintainable edits
  - `reviewer-opus46` — `github-copilot/claude-opus-4.6`; read-only review

## Practice Repo

- Local path: `/home/marat/dev/git/markul/opencode`
- Structure:
  - `opencode.json` — project config; loads `docs/instructions.md` via `instructions` field
  - `docs/instructions.md` — shared coding guidelines loaded into every session
  - `docs/agents.md` — full reference: agent types, config options, role patterns
  - `.opencode/agents/reviewer.md` — read-only code review subagent
  - `.opencode/agents/planner.md` — planning subagent; bash read-only (git/grep/find)
- Next major step: add the orchestrator primary inside the practice repo and validate the delegation flow against the global setup
