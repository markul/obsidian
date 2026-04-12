---
tags:
  - agents/project
  - status/active
---

# UTK PWR

## Status

- `active`

## Timestamps

- Created: `2026-04-01 13:30`
- Completed: `not completed`

## Goal

- Create an agent skill that monitors configured objects in Bitbucket pull requests
- Accept an `SKPDEPLOY` Jira ticket such as `SKPDEPLOY-12033` as input and return monitored objects found across the relevant pull requests

## Plan

1. ~~Define the exact `utk-mcp` calls needed for direct-artifact traversal and linked-issue traversal.~~
2. ~~Define the normalization and matching strategy for monitored objects versus PR or compare-diff content.~~
3. ~~Add representative test cases for both collection paths.~~
4. ~~Handle `compare/commits` URL variant in `check-pwr-by-artifacts`.~~ ✔
5. Fix inventory typo `skp-restructing-service` → `skp-restructuring-service` in the Confluence xlsx source (or fix directly in `pwr-list.json` until xlsx is updated).
6. Decide whether Phase 2 runs after a Phase 1 hit (needed for mixed API+DB tickets like `SKPDEPLOY-9257`, `SKPDEPLOY-10694`).
7. Fix or replace `check-pwr-by-linked-tickets` — blocked by `utk-mcp` connector limitation for non-SKPDEPLOY project keys.

## Scope

- Local path: `/home/marat/dev/git/alfa/utk-pwr`
- Main monitored-objects directory: `/home/marat/dev/git/alfa/utk-pwr/objects-to-monitor`
- Integration boundary: all Jira and Bitbucket interaction must go through `utk-mcp`
- Related notes:
  - [[daily/2026-04-01]]
  - [[agents/sessions/2026-04-01-utk-pwr-agent-project]]

## Validation

## Current State

- Monitoring inventory sourced from Confluence xlsx (pageId=457221253) via `$update-pwr-list`; 69 objects across 7 groups in `pwr/pwr-list.json`
- `pwr/` directory is gitignored (generated/downloaded files); `README.md` added
- `update-pwr-list` tracks **attachment version** (not page version) in `pwr/pwr-version.json`; 5-step workflow using `list_confluence_page_attachments` as the single version-check call
- Both check skills always invoke `$update-pwr-list` first with transparent Russian-language version log
- `check-pwr-by-artifacts` handles `compare/diff` and `compare/commits` URLs; normalizes `compare/commits` → `compare/diff` before calling `get_bitbucket_compare_diff`
- Two-phase matching: Phase 1 checks repo slug against service set (no API); Phase 2 fetches content only on Phase 1 miss
- Live test results: `SKPDEPLOY-6185` ✔, `SKPDEPLOY-9257` partially ✔ (`ufr-coa-collateralobjectgroup-service` now found; `ufr-coa-shared-nuget` and `ufr-lm-restructuring-service` still missed due to Phase 2 skip)
- `check-pwr-by-linked-tickets`: blocked — `get_jira_development_metadata` errors for non-SKPDEPLOY keys
- Related sessions:
  - [[agents/sessions/2026-04-06-utk-pwr-skill-refinement]]
  - [[agents/sessions/2026-04-07-utk-pwr-readme-and-cleanup]]

## Key Decisions

- Keep this work scoped around the local `utk-pwr` workspace even though it is not currently a Git checkout
- Treat `utk-mcp` as the only integration layer for Jira and Bitbucket access
- Support two collection paths: pull requests linked directly to the input `SKPDEPLOY` ticket and pull requests linked to related non-`SKPDEPLOY` Jira issues
- Use the actual local directory name `objects-to-monitor/`

## Blockers

- `skp-restructing-service` inventory typo — `SKPDEPLOY-13680` produces only a typo note until fixed in Confluence xlsx
- Phase 2 skipped after Phase 1 hit — mixed API+DB tickets (`SKPDEPLOY-9257`, `SKPDEPLOY-10694`) miss objects found only in diff content
- `get_jira_development_metadata` errors for non-SKPDEPLOY project keys — fully blocks `check-pwr-by-linked-tickets`
