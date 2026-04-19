---
tags:
  - agents/project
note-type: agent-project
status: active
created: 2026-04-12T06:20
completed: 
related-ticket: 
---

# Second Brain

## Status

- `active`

## Timestamps

- Created: `2026-04-12 06:20`
- Completed: `not completed`

## Goal

- Improve this Obsidian vault into a durable second-brain system for AI-assisted learning, personal knowledge, and work retrieval
- Establish a default agent workflow where meaningful work triggers a brief Obsidian-update check

## Plan

1. Refine a lightweight linking and retrieval model that works with the existing folder structure.
2. Grow durable personal AI learning notes and topic hubs so useful ideas are easier to rediscover.
3. Promote reusable lessons from work, sessions, and agent projects into evergreen personal notes.

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`
- Hub note: [[personal/learning/ai/index|AI]]
- Learning note: [[personal/learning/ai/second-brain|Second Brain]]
- Related notes:
  - [[daily/2026-04-12]]
  - [[agents/sessions/2026-04-12-second-brain-kickoff]]
  - [[agents/sessions/2026-04-12-second-brain-resume]]
  - [[agents/projects/opencode|OpenCode Multi-Agent]]

## Validation

## Current State

- The vault now has initial retrieval hubs for AI, Tech, and Alfa Bank.
- `AGENTS.md` now includes a lightweight linking-and-retrieval policy.
- Shared agent guidance in `/home/marat/.agents/AGENTS.md` now tells agents to consider Obsidian updates by default for meaningful work and to do a brief end-of-task note check.
- The detailed note-update workflow remains in the vault-local `AGENTS.md`, keeping the shared rule short and the vault rule specific.
- The initial retrieval and linking experiment is complete.
- Resumed 2026-04-12: next step is to turn second-brain ideas into durable personal notes and repeatable note-maintenance habits.
- Reviewed [[personal/learning/ai/docs/llm-wiki|LLM Wiki]] as a related pattern; some ideas may be useful in a future redesign, but no structural changes were adopted yet.
- Obsidian Properties adoption: the vault now uses properties for work tickets, agent projects, and session notes as small, stable metadata (note-type, status, dates, relations); properties supplement markdown sections without replacing them.
- Removed `status/...` tags from all frontmatter; `status` property is now the canonical home for project status.
- Added `alfa-bank/ticket` tag to all Alfa ticket notes (6 files) and the ticket template for broad filtering across Alfa work; documented in `AGENTS.md` Work Tickets section.
- Alfa ticket notes now use a single stable `work/alfa-bank/tickets/` directory; local status lives in frontmatter and the Alfa index groups tickets by status instead of encoding status in the path.

## Key Decisions

- Keep the existing top-level vault structure and improve retrieval with links, hub notes, and evergreen personal notes instead of a broad reorganization.
- Treat `personal/` as the main long-term knowledge layer, while `agents/` and `work/` remain the operational context.
- Adopt Obsidian Properties for work tickets, agent projects, and session notes to add small, stable metadata; properties supplement markdown sections and do not replace them.
- Keep work ticket paths stable and avoid folder-based status encoding when frontmatter plus an index already captures the state cleanly.
- Future optimization: if the properties pattern remains useful, reduce duplicated markdown metadata such as `Status`, `Timestamps`, and repeated Jira fields so properties become the canonical home for small stable facts.

## Blockers

- None.
