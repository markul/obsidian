---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-24
service:
project: "[[agents/projects/second-brain|Second Brain]]"
ticket:
---

# 2026-04-24 obsidian-vault-structure-review

## Goal

- Review the vault structure and representative notes for structural drift against the current `AGENTS.md` policy

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`
- Related notes:
  - [[daily/2026-04-24]]
  - [[agents/sessions/2026-04-14-obsidian-vault-repo-analysis]]
  - [[agents/sessions/2026-04-14-context-budget-audit]]

## Actions

- Reviewed the vault policy in `AGENTS.md` and the local `obsidian-vault-workflow` skill
- Sampled current project, ticket, session, template, and daily notes
- Ran lightweight consistency checks across `agents/projects/`, `agents/sessions/`, `work/alfa-bank/tickets/`, and `daily/`
- Identified structural drift in legacy session notes, a supporting project note stored in the canonical project directory without machine-readable metadata, an out-of-date Alfa ticket index, and daily notes that exceed the intended one-line chronology style

## Decisions

- Keep this review as a session note only; no durable project note is needed for a one-pass vault audit
- Treat the most important follow-up as structural normalization, not content rewriting

## Follow Up

- Normalize legacy session-note frontmatter and section shape in small batches
- Decide whether supporting notes under `agents/projects/` need a lighter explicit metadata pattern or a different folder
- Trim future daily notes back to one-line chronology with links to project or session notes
