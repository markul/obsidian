---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-01
service: "[[work/alfa-bank/services/utk-pwr|utk-pwr]]"
project: "[[agents/projects/utk-pwr|UTK PWR]]"
ticket:
---

# 2026-04-01 utk-pwr-agent-project

## Goal

- Create a stable Obsidian agent project for `utk-pwr`
- Capture the initial requirements for the planned monitoring skill

## Scope

- Repo: this Obsidian vault
- Related notes:
  - [[daily/2026-04-01]]
  - [[agents/projects/utk-pwr|UTK PWR]]
  - [[agents/projects/utk-pwr|UTK PWR]]

## Actions

- Read the vault workflow skill, `AGENTS.md`, and the project/session checklists
- Inspected existing agent project, daily note, and session-note patterns
- Verified the requested local path at `/home/marat/dev/git/alfa/utk-pwr`
- Observed that the path is not a Git worktree and contains test/data-oriented files and directories
- Created `agents/projects/utk-pwr/` with `project.md`, `plan.md`, and `result.md`
- Linked the new project from [[daily/2026-04-01]]
- Verified that the monitored-object directory is `objects-to-monitor/`, not `objects-to-monitore/`
- Captured the skill goal: given an `SKPDEPLOY` ticket, inspect Bitbucket pull requests and return the monitored objects found
- Captured the two pull-request collection cases: direct `SKPDEPLOY` PRs and PRs linked to non-`SKPDEPLOY` issues related to the input ticket
- Recorded that all Jira and Bitbucket interaction must go through `utk-mcp`
- Updated the direct-artifact skill so Jira field `Артефакты поставки` may contain Bitbucket pull request links or Bitbucket compare-diff links
- Recorded that compare-diff links must be resolved through `get_bitbucket_compare_diff`
- Updated the direct-artifact skill to call out possible inventory typos when artifact service names nearly match inventory entries but do not match exactly
- Updated the linked-ticket skill to use the same possible-inventory-typo note rule for near-miss service names
- Updated the linked-ticket skill so linked-issue pull requests should be sourced through `get_jira_development_metadata`

## Decisions

- Use `agents/projects/utk-pwr/` as the stable workspace for this effort
- Promote the project status to `active` now that the skill scope is defined
- Use the actual local directory name `objects-to-monitor/`
- Treat `utk-mcp` as the sole integration boundary for Jira and Bitbucket operations
- The final skill output is the list of monitored objects found in the analyzed pull requests
- The direct-artifact skill must support both pull-request and compare-diff links from Jira custom fields
- Possible inventory typos should be reported as notes, not promoted to found matches
- Both the direct-artifact and linked-ticket skills should apply the same possible-inventory-typo rule
- The linked-ticket skill should source pull requests from linked Jira issues through `get_jira_development_metadata`

## Follow Up

- Define the concrete `utk-mcp` calls needed to traverse linked Jira issues and direct Bitbucket artifacts
- Define the matching rules for monitored objects across compare diffs, PR changed files, or both
