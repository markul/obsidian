---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-25
service:
project: "[[agents/projects/my-infra|My Infra]]"
ticket:
---

# Markul Docker Review

## Goal

- Review the latest changes in `/home/marat/dev/git/markul/docker`.

## Scope

- Related project: [[agents/projects/my-infra|My Infra]]
- Repository: `/home/marat/dev/git/markul/docker`
- Reviewed change set: commit `b941266`

## Actions

- Checked the repo worktree state and confirmed it was clean.
- Reviewed the latest commit stat and full diff for `b941266`.
- Compared the new Helsinki `nginx-proxy` files against the existing `dev.markul.net` versions.
- Pulled exact line references for the new Optiplex `seerr`, Kafka, and registry-cleanup changes.

## Findings

- `docker-compose/optiplex.markul.net/kafka/docker-compose.yml` advertises Kafka as `kafka:9092`, so published host port `9092` is misleading for host or LAN clients; only containers on the Compose network can use that broker address.
- `docker-compose/infra.markul.net/docker-registry/registry_cleanup.sh` only iterates repository paths as `repo/image` and breaks on root-level repositories or deeper nested names, so cleanup coverage is incomplete and path assumptions are fragile.
- `docker-compose/optiplex.markul.net/jellyfin/docker-compose.yml` pins `seerr` to public DNS `9.9.9.9` and compensates with static `extra_hosts`, which means any future internal service hostname still has to be hardcoded manually.

## Decisions

- Treat the review as actionable infra follow-up rather than note-only work.

## Validation

- `git -C /home/marat/dev/git/markul/docker status --short`
- `git -C /home/marat/dev/git/markul/docker show --stat --summary b941266`
- `git -C /home/marat/dev/git/markul/docker diff f1ee2df..b941266`

## Outcome

- Review completed with concrete findings and line references; no code changes were made in the Docker repo during this pass.
