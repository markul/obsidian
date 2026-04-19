# Obsidian ticket instructions update

## Scope

- Vault: `/home/marat/dev/git/markul/obsidian`

## Goal

- Align the vault rules and the `obsidian-vault-workflow` skill with the current single-file ticket model and the `UTK2-3264` example

## Actions

- Updated `AGENTS.md` to codify single-file work tickets under `tickets/{status}/{ticket}.md`
- Added explicit work-ticket section order guidance: `Status`, `Goal`, `Jira Snapshot`, `Development Metadata`, `Notes`
- Marked [[work/alfa-bank/tickets/utk2-3264|UTK2-3264]] as the canonical ticket example
- Clarified that `Development Metadata` should contain only durable tracking facts
- Clarified that intermediate implementation details and detailed validation belong in agent project and session notes
- Updated the `obsidian-vault-workflow` skill to match the same ticket model and examples

## Decision

- Agents should follow the `UTK2-3264` ticket shape strictly unless a user explicitly asks for a different ticket format
