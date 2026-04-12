# AGENTS

## Purpose

This repository is an Obsidian vault used to organize personal notes, work notes, and AI-assisted research or coding sessions.

## Vault Structure

- `agents/`: shared area for agent workflows, sessions, prompts, templates, and agent-specific subfolders
- `agents/{agent}/`: per-agent notes, preferences, or experiments that should not be mixed into shared notes
- `agents/projects/{name}.md`: stable agent project notes
- `agents/prompts/`: reusable prompts and prompt patterns
- `agents/sessions/`: meaningful agent session records
- `agents/templates/`: note templates for projects, sessions, and tasks
- `daily/`: daily notes and lightweight chronological logs, typically as `YYYY-MM-DD.md`
- `personal/tech/`: personal infrastructure, hardware, software inventory, upgrade planning, and benchmarks
- `work/`: employer or client-specific notes and durable project context

## Editing Rules

- Preserve Obsidian wiki links
- Prefer editing existing notes over creating duplicates
- Use clear note titles instead of date-only titles unless the note is a session log
- Avoid spaces in file and directory names; prefer kebab-case such as `project-template.md`
- Keep folder names stable so internal links do not break
- Default to Markdown; avoid adding tooling or app scaffolding unless explicitly requested

## Note Placement

- Put raw ideas and follow-ups in notes under `agents/inbox/`
- Put ongoing multi-session work in `agents/projects/{name}.md`
- Put reusable prompts or instructions in `agents/prompts/`
- Put session notes in `agents/sessions/`
- Put agent-specific notes in `agents/{agent}/`
- Put daily notes in `daily/`
- Put personal technical reference material under `personal/tech/`, including machine software inventory notes such as `personal/tech/software.md`
- Put employer or client-specific material under `work/`
- Treat Jira issues and Confluence work items as work-related by default, not personal
- Under each employer or client area in `work/`, keep tickets under `tickets/new/`, `tickets/active/`, `tickets/on-hold/`, and `tickets/done/`, with one Markdown file per ticket
- Ticket notes should live directly at paths such as `work/alfa-bank/tickets/active/utk2-3264.md`

## Workspace Guidance

### General

- Before making broad changes, read `AGENTS.md` and inspect the relevant folders and nearby durable notes
- If you add a substantial new note, link it from the nearest related durable note when useful
- Do not rewrite note organization unless asked
- Treat this repo as documentation-first: optimize for clarity, stable links, and minimal churn
- Do not rely on `Home.md` index notes; this vault now uses folder structure and direct note links instead
- Keep instructions tool-agnostic unless a note is explicitly dedicated to one workflow

### Agent Projects

- For agent projects, use a single stable note at `agents/projects/{name}.md`
- The canonical durable state for an agent project should stay in that single stable note even when supporting notes also exist
- If a plan or reference grows too large for the canonical project note, it is acceptable to create a clearly named supporting note such as `agents/projects/{name}-plan.md`
- Supporting notes must supplement, not replace, the canonical project note, and the canonical note should link to them when they matter to current work
- For agent projects, track status with tags such as `#status/new`, `#status/active`, `#status/on-hold`, or `#status/done`
- In each agent project note, include `Created` and `Completed` timestamps with date and time to the minute
- Keep agent project notes concise and durable; do not turn them into step-by-step work logs
- When a work ticket also exists, treat the agent project note as the main durable home for active implementation context, including current state, plan, key decisions, blockers, and validation commands
- Agent project notes are expected to preserve resumable context across sessions by default; do not restate that as part of an individual project's `Goal`
- Use this section order for agent projects unless there is a strong reason not to:
  - `Status`
  - `Timestamps`
  - `Goal`
  - `Plan`
  - `Scope`
  - `Validation`
  - `Current State`
  - `Key Decisions`
  - `Blockers`
- Keep `Validation` optional
- When present, keep `Validation` limited to the minimal required commands needed to verify the project state; do not put validation results there
- For broad projects without a clear validation path, leave `Validation` empty instead of inventing weak commands
- Keep `Plan` as a short numbered list of the current intended path
- Keep `Blockers` limited to real current blockers; remove them once they are no longer blocking
- Keep shared note-placement, durability, and workflow requirements in `AGENTS.md`, not in project-specific `Key Decisions`
- Keep project `Key Decisions` limited to project-specific technical or operational decisions
- When a work ticket moves to local `done`, move the corresponding agent project to `#status/done` and set its `Completed` timestamp

### Daily Notes And Sessions

- Keep daily notes minimal: each work item should usually be a single-line bullet with a short summary and a link to the related agent project or ticket
- For meaningful agent work, automatically create or update a session note under `agents/sessions/`
- Name session notes as `YYYY-MM-DD-topic.md`
- Every session note should link to the related agent project in its `Scope` or related-notes section when an agent project exists
- Use session notes to capture the detailed chronology, commands, decisions, blockers, validation, and handoff context needed for another Codex run to resume the work

### Alfa Work

- For Alfa internal resources, follow the `alfa-internal-mcp` skill and use `utk-mcp` only; do not use fallback access methods
- Before Alfa internal requests, verify `snxctl status` and confirm the VPN is connected; if not, report the limitation and stop
- Treat VPN and MCP access as shared Alfa-work preconditions, not as project-specific `Validation`

### Work Tickets

- Each work ticket note should include a clickable Markdown link to the Jira issue
- Each work ticket note should use an H1 header that contains both the ticket key and the Jira summary
- Each work ticket note may include the remote Bitbucket repository link and the local repository path when they are known
- Each work ticket note should include a `Development Metadata` section for branches, pull requests, repositories, builds, or other implementation-tracking facts when available
- Each work ticket note should follow this section order unless there is a strong reason not to: `Status`, `Goal`, `Jira Snapshot`, `Development Metadata`, `Notes`
- Treat [[work/alfa-bank/tickets/active/utk2-3264|UTK2-3264]] as the canonical example for work ticket structure
- Keep `Development Metadata` limited to durable implementation-tracking facts such as repository, local path, agent project, branch, commit, pull request, and build
- Keep work ticket notes minimalistic
- Put current implementation state, expected implementation path, schema or technical decisions, blockers, experiments, and detailed validation chronology into the related agent project and session notes instead of the ticket note
- In the ticket note `Notes` section, prefer a short pointer to the related agent project over repeating active working context
- When Jira tickets include links, store them as clickable Markdown links with the full usable URL in the target, not partial ids or fragments; for Confluence pages, fetch the page title and use it as the link text
- Do not restate that a note is a Jira work item inside `work/.../tickets/...`; that is implied by its location
- For internal vault navigation to ticket notes, prefer short wiki-link aliases such as `[[work/alfa-bank/tickets/active/utk2-3264|UTK2-3264]]`
- When a work ticket changes local status, move the single ticket file to the matching status folder and record the change in the relevant daily note
- When a work ticket moves to local `done`, update the linked agent project in `agents/projects/` to `done` in the same pass
