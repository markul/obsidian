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
- Under each employer or client area in `work/`, keep ticket notes in one stable `tickets/` directory, with one Markdown file per ticket
- Ticket notes should live directly at paths such as `work/alfa-bank/tickets/utk2-3264.md`

## Workspace Guidance

### General

- Before making broad changes, read `AGENTS.md` and inspect the relevant folders and nearby durable notes
- If you add a substantial new note, link it from the nearest related durable note when useful
- Do not rewrite note organization unless asked
- Treat this repo as documentation-first: optimize for clarity, stable links, and minimal churn
- Do not rely on `Home.md` index notes; this vault now uses folder structure and direct note links instead
- Keep instructions tool-agnostic unless a note is explicitly dedicated to one workflow

### Linking And Retrieval

- Use folders for stable placement and links for meaning; do not rely on folder location alone for retrieval
- Prefer direct wiki links between genuinely related notes across `personal/`, `work/`, `agents/`, and `daily/`
- It is acceptable to create small topic hub notes such as `index.md` in a relevant area when they improve navigation
- When a durable note has several important neighbors, add a short `Related Notes` section or equivalent local links
- When work or agent notes produce reusable knowledge, promote that understanding into a durable personal note and link the notes when useful

### Agent Projects

- For agent projects, use a single stable note at `agents/projects/{name}.md`
- The canonical durable state for an agent project should stay in that single stable note even when supporting notes also exist
- If a plan or reference grows too large for the canonical project note, it is acceptable to create a clearly named supporting note such as `agents/projects/{name}-plan.md`
- Supporting notes must supplement, not replace, the canonical project note, and the canonical note should link to them when they matter to current work
- Agent project notes now use Obsidian Properties in frontmatter for small, stable metadata such as `note-type`, `status`, `created`, `completed`, and `related-ticket`
- Properties supplement markdown sections; do not remove sections like `Status`, `Timestamps`, `Goal`, `Scope`, etc.
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
- When a work ticket moves to local `done`, update the corresponding agent project `status` property to `done` and set its `Completed` timestamp

### Daily Notes And Sessions

- Keep daily notes minimal: each work item should usually be a single-line bullet with a short summary and a link to the related agent project or ticket
- For meaningful agent work, automatically create or update a session note under `agents/sessions/`
- Name session notes as `YYYY-MM-DD-topic.md`
- Session notes now use Obsidian Properties in frontmatter for small, stable metadata such as `note-type`, `session-date`, `related-project`, and `related-ticket`
- Properties supplement markdown sections; do not remove sections like `Goal`, `Scope`, `Actions`, `Decisions`, etc.
- Every session note should link to the related agent project in its `Scope` or related-notes section when an agent project exists
- Use session notes to capture the detailed chronology, commands, decisions, blockers, validation, and handoff context needed for another Codex run to resume the work

### Context Budget

- Update notes when doing so will reduce future re-reading or re-discovery work for another agent
- Prefer the smallest durable note update that preserves resumable context
- Put current truth in the agent project note, not in the daily note
- Put detailed chronology for the current run in the session note, not in the agent project note
- Put only a one-line chronological trace in the daily note unless more detail is genuinely needed
- Put only durable metadata and short pointers in work ticket notes; avoid copying active implementation context there
- Do not repeat the same status or decision across project, session, daily, and ticket notes unless each copy serves a distinct retrieval purpose
- If a project section starts reading like a work log, move that material into a session note
- Summarize logs, command output, and diagnostics instead of pasting long raw output unless exact text is required for future debugging
- Skip note updates for trivial one-off work that has no likely future retrieval value

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
- Work ticket notes now use Obsidian Properties in frontmatter for small, stable metadata such as `note-type`, `ticket-key`, `status`, `jira-status`, `created`, `updated`, `assignee`, and `agent-project`
- Alfa Bank ticket notes should include the `alfa-bank/ticket` tag in frontmatter for broad filtering across all Alfa work tickets
- Properties supplement markdown sections; do not remove sections like `Status`, `Goal`, `Jira Snapshot`, etc.
- Treat [[work/alfa-bank/tickets/utk2-3264|UTK2-3264]] as the canonical example for work ticket structure
- Keep `Development Metadata` limited to durable implementation-tracking facts such as repository, local path, agent project, branch, commit, pull request, and build
- Keep work ticket notes minimalistic
- Put current implementation state, expected implementation path, schema or technical decisions, blockers, experiments, and detailed validation chronology into the related agent project and session notes instead of the ticket note
- For active tickets, it is acceptable to keep one short durable current-state bullet in `Notes` so the next reader can see the local standing immediately without opening the project note
- In the ticket note `Notes` section, prefer a short pointer to the related agent project over repeating active working context
- When Jira tickets include links, store them as clickable Markdown links with the full usable URL in the target, not partial ids or fragments; for Confluence pages, fetch the page title and use it as the link text
- Do not restate that a note is a Jira work item inside `work/.../tickets/...`; that is implied by its location
- For internal vault navigation to ticket notes, prefer short wiki-link aliases such as `[[work/alfa-bank/tickets/utk2-3264|UTK2-3264]]`
- Keep ticket file paths stable when local status changes; update the ticket `status` property and the relevant index note instead of moving the file
- When a work ticket moves to local `done`, update the linked agent project in `agents/projects/` to `done` in the same pass
