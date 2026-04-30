---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-29
service: "[[work/alfa-bank/services/skp-product-change-workflow-service|skp-product-change-workflow-service]]"
related-project: "[[agents/projects/utk2-3339|UTK2-3339 Agent Work]]"
related-ticket: "[[work/alfa-bank/tickets/utk2-3339|UTK2-3339]]"
---

# 2026-04-29 utk2-3339-skaffold-instruction

## Goal

- Start `UTK2-3339` and capture the initial Jira, Confluence, repo, and local-note context for preparing the Skaffold instruction.

## Scope

- Repo: `/home/marat/dev/git/alfa/skp-product-change-workflow-service`
- Jira: [UTK2-3339](https://jira.moscow.alfaintra.net/browse/UTK2-3339)
- Confluence draft: [\[ЧЕРНОВИК\] Как добавить Skaffold](https://confluence.moscow.alfaintra.net/spaces/LOANMGR/pages/3138325534/%D0%A7%D0%95%D0%A0%D0%9D%D0%9E%D0%92%D0%98%D0%9A+%D0%9A%D0%B0%D0%BA+%D0%B4%D0%BE%D0%B1%D0%B0%D0%B2%D0%B8%D1%82%D1%8C+Skaffold)
- Related notes:
  - [[agents/projects/utk2-3339|UTK2-3339]]
  - [[work/alfa-bank/tickets/utk2-3339|UTK2-3339]]
  - [[daily/2026-04-29]]

## Actions

- Verified Alfa VPN connectivity with `snxctl status`.
- Queried Jira issue `UTK2-3339` through `utk-mcp`.
- Confirmed Jira status is `Backlog`, assignee is `Кульмухаметов Марат Мидхатович`, and the request is to update the Skaffold instruction.
- Queried Jira remote links and development metadata; no remote links, repositories, branches, or pull requests are currently linked to the issue.
- Resolved the Confluence page from the Jira description by page id `3138325534`; title is `[ЧЕРНОВИК] Как добавить Skaffold`, space is `LOANMGR`, version is `7`.
- Checked local repo state: branch `integration-2`, remote `ssh://git@git.moscow.alfaintra.net/ufrkpnsb/skp-product-change-workflow-service.git`, and no uncommitted source changes.
- Read the repo's current `skaffold.yaml`, `devops/base`, `devops/dependencies`, `devops/api`, integration-test Dockerfile, `appsettings.Local.yaml`, README Skaffold section, and Jenkins integration-test runner.
- Created local draft `docs/skaffold-generic-service-instruction.md` with a generic service instruction following the Confluence page structure and using this repo's implementation as the reference model.
- Checked that Markdown code fences in the new document are balanced.
- Created `docs/skaffold-generic-service-confluence.txt` as a Confluence wiki-markup copy/paste version using `h1.`, `h2.`, `{{inline code}}`, and `{code:language=...}` blocks.
- Checked that Confluence `{code}` macros in the copy/paste document are balanced.
- Added explicit Jenkins pipeline guidance to both local documents, covering `devops/Jenkinsfile`, `devops/RunTests.groovy`, `pipeline-settings.json`, and required Pod annotations for Docker images and the test container.

## Decisions

- Start from the current Confluence draft and compare it against the latest working `skp-product-change-workflow-service` Skaffold guidance before proposing page edits.
- Track active implementation and instruction-analysis state in [[agents/projects/utk2-3339|UTK2-3339 Agent Work]].

## Validation

- `python3` Markdown fence balance check for `docs/skaffold-generic-service-instruction.md`
- `python3` Confluence `{code}` macro balance check for `docs/skaffold-generic-service-confluence.txt`

## Follow Up

- Review the local draft and decide which sections should be copied into the Confluence page.
- If the Confluence page will be updated directly, preserve the current section shape but replace older standalone `skaffold/` examples with the root `skaffold.yaml` plus Kustomize overlay model.
