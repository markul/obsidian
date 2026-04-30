---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-13
service: "[[work/alfa-bank/services/ufr-kpnsb-limit-service|ufr-kpnsb-limit-service]]"
related-project: 
related-ticket: 
---

# 2026-04-13 ufr-kpnsb-limit-service-flaky-test-stash

## Goal

- Preserve the context for a flaky `ufr-kpnsb-limit-service` unit-test fix that was stashed instead of committed.

## Scope

- Repo: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Related notes:
  - [[daily/2026-04-13]]

## Actions

- Investigated the flaky unit test `ProcessLimitGroupAsync_ShouldCreateChangeLimitRequest_WithSubscription`.
- Narrowed the likely cause to random mock setup collisions between `LmLimitId` and `LmSublimitId`.
- Saved the proposed fix as Git stash `stash@{0}` with label `fix-failing-test-randomly`.
- Recorded the retrieval commands needed to inspect and re-apply the stash later.

## Decisions

- Keep the proposed fix as a stash instead of committing it until the team decides whether to revive or revise the approach.

## Follow Up

- Retrieval commands:

  ```bash
  cd ~/dev/git/alfa/ufr-kpnsb-limit-service
  git stash list --grep='fix-failing-test-randomly'
  git stash show -p stash@{0}
  git stash apply stash@{0}
  ```
