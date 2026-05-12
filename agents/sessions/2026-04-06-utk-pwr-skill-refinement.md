---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-06
service: "[[work/alfa-bank/services/utk-pwr|utk-pwr]]"
project: "[[agents/projects/utk-pwr|UTK PWR]]"
ticket:
---

# 2026-04-06 utk-pwr skill refinement

## Goal

- Analyze the `utk-pwr` workspace and both skills
- Identify inventory and connector gaps
- Fix skill definitions based on live test runs against all `test-cases/case-data.md` cases

## Scope

- Repo: `/home/marat/dev/git/alfa/utk-pwr`
- Related notes:
  - [[agents/projects/utk-pwr|UTK PWR]]

## Actions

- Analyzed workspace: two skills (`check-pwr-by-artifacts`, `check-pwr-by-linked-tickets`), two CSV inventories, no `.txt` files on disk
- Removed `.txt` file references from both skill SKILL.md inventory loaders; both now source exclusively from `objects-to-monitor/*.csv`
- Identified inventory gaps: `loanmanager-questionnaire-service`, `skp-restructuring-service` (typo in inventory as `skp-restructing-service`), `wfCreateRequest`, `lm-problem-service` present in CSVs but were missing from assumed `.txt` sources
- Created `skills/create-pwr-list/` skill with `SKILL.md`, `generate.py`, and `agents/openai.yaml`; script parses both CSVs, groups objects by type, writes `pwr-list.json` to repo root
- Generated `pwr-list.json`: 69 objects across 7 groups (`skp-бд-table-ft-rl`, `skp-бд-table-dr`, `skp-бд-view`, `skp-бд-procedure`, `skp-бд-table`, `skp-api`, `ufr-api`)
- Trimming warnings caught: `SubjectRisk_Level_Set`, `ufr-coa-collateralobjectgroup-service`, `ufr-coa-worksheet-producer` had trailing whitespace in CSVs
- Updated both check skills to source monitoring set from `pwr-list.json` instead of raw CSVs
- Added two-phase matching to both check skills: Phase 1 checks repo slug / URL metadata against the service set (API groups) — no API call; Phase 2 fetches PR/diff content only when Phase 1 misses
- Added `pwr-list.json` existence check to both check skills: invoke `$create-pwr-list` if missing

### `check-pwr-by-artifacts` test run (all 10 test cases)

- Root cause of prior failures: `get_jira_issue` must be called with `customFields: ["Ссылки на ПР-ы", "Артефакты поставки"]` — fields are empty without this
- Fixed Step 2 in SKILL.md to document the exact `customFields` parameter usage
- Results with correct custom field param:
  - `SKPDEPLOY-12033` → `ufr-lm-problem-service` ✓ (Phase 1, compare-diff slug match)
  - `SKPDEPLOY-13328` → `ufr-skp-mdc-service` ✓ (Phase 1, compare-diff slug match)
  - `SKPDEPLOY-10423` → `ufr-lm-restructuring-service` ✓ (Phase 1, compare-diff slug match)
  - `SKPDEPLOY-12598` → `SOK_UpdateSubjectRating` ✓ (Phase 2, PR changed file path)
  - `SKPDEPLOY-6185` → `CollateralRisk_set` ✓ (Phase 2, PR changed file path)
  - `SKPDEPLOY-13556` → `wfCreateRequest` ✓ (Phase 2, PR changed file path)
  - `SKPDEPLOY-13680` → typo note only; inventory has `skp-restructing-service`, artifact repo is `skp-restructuring-service` (missing `r`)
  - `SKPDEPLOY-13332` → `ufr-lm-restructuring-service` ✓; `loanmanager-questionnaire-service` missing (no artifact URL points to it)
  - `SKPDEPLOY-10694` → `ufr-lm-problem-service` ✓ via Phase 1; `lm-problem-service` missed because Phase 2 skipped after Phase 1 hit
  - `SKPDEPLOY-9257` → `ufr-coa-reservegroup-service` via Phase 1 (unexpected); `ufr-coa-collateralobjectgroup-service` missed because first artifact URL uses `compare/commits` (not `compare/diff`) which the parser skips

### `check-pwr-by-linked-tickets` test run

- `get_jira_development_metadata` errors for every non-SKPDEPLOY project key (BAURISKS-*, PREF-*, COA-*, FM1-*, OVERMSB-*, CTRLRISKS-*, etc.)
- Linked SKPDEPLOY-* tickets that do respond return zero PRs (they are platform delivery tickets, not code tickets)
- All 10 test cases blocked — skill cannot produce results until `utk-mcp` supports dev metadata for non-SKPDEPLOY project keys, or an alternative PR source is used

## Decisions

- Both check skills use `pwr-list.json` as inventory source with auto-invoke of `$create-pwr-list` if missing
- Two-phase matching is the correct approach for efficiency; Phase 2 skip on Phase 1 hit is a known limitation for tickets with multiple object types
- `check-pwr-by-linked-tickets` is blocked by a `utk-mcp` connector limitation

## Blockers (at session start)

- `skp-restructing-service` inventory typo must be fixed in `skp.csv` to resolve `SKPDEPLOY-13680`
- `get_jira_development_metadata` does not work for non-SKPDEPLOY issue keys — blocks all linked-ticket traversal
- `compare/commits` URL variant not handled — blocks `SKPDEPLOY-9257` artifact 1
- Phase 2 skip after Phase 1 hit misses additional object types on same artifact — affects `SKPDEPLOY-10694`

---

## Continuation — version tracking, compare/commits, gitignore

### Version tracking: attachment version instead of page version

- `update-pwr-list` was tracking the Confluence **page** version in `pwr-version.json` (`confluence_version` key)
- Confluence page version changes on any edit of the page body/metadata, not just when the xlsx attachment changes
- Fixed: version tracking now uses the **attachment version** (`attachment_version` key) from `list_confluence_page_attachments`
- `update.py`: renamed argument and variable `confluence_version` → `attachment_version` throughout; `pwr-version.json` and `pwr-list.json` now use `attachment_version` key
- `SKILL.md` workflow restructured from 6 steps to 5 steps:
  - Step 1: `list_confluence_page_attachments` (gets both filename and attachment version — replaces separate `get_confluence_page` call)
  - Step 2: compare `attachment_version` from local `pwr-version.json`
  - Step 3: download (was Step 4)
  - Step 4: parse (was Step 5)
  - Step 5: report (was Step 6)
- Saved one MCP call per run when file is up to date
- Re-ran `update.py` with `attachment_version=3` to fix current `pwr-version.json` and `pwr-list.json`

### compare/commits support

- Bitbucket URLs in `Артефакты поставки` can use either `/compare/diff` or `/compare/commits` — both carry identical query parameters (`sourceBranch`, `targetRepoId`, `targetBranch`)
- `SKPDEPLOY-9257` artifact 1 uses `compare/commits`; it was previously skipped entirely (not even Phase 1 applied)
- Fixed in `check-pwr-by-artifacts/SKILL.md`:
  - URL regex broadened to match `compare/(diff|commits)`
  - Normalization rule added: replace `/compare/commits` with `/compare/diff` before calling `get_bitbucket_compare_diff`
  - Phase 1 and Phase 2 descriptions updated to say "compare URL" instead of "compare-diff"
  - Guardrail added: normalize before calling the API
- Re-tested `SKPDEPLOY-9257`: artifact 1 now recognized and matched via Phase 1 (`ufr-coa-collateralobjectgroup-service`) ✓
- Remaining gap: `ufr-coa-shared-nuget` and `ufr-lm-restructuring-service` appear in diff **content** of URLs 1 and 2, but Phase 1 matches those artifacts on their repo slug and skips Phase 2 — content not scanned

### .gitignore

- Created `.gitignore` at repo root with `pwr/` — generated/downloaded files should not be committed

## Follow Up

- Fix `skp-restructing-service` → `skp-restructuring-service` in `skp.csv` and regenerate `pwr-list.json`
- Handle `compare/commits` URL variant alongside `compare/diff` in artifact skill
- Decide whether Phase 2 should still run after a Phase 1 hit (to catch mixed API + DB tickets)
- Investigate `utk-mcp` fix for non-SKPDEPLOY dev metadata, or redesign linked-ticket skill to use alternate PR source
