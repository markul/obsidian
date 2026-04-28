---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-26
related-project:
  - "[[agents/projects/my-infra|My Infra]]"
related-ticket:
---

# markul.arm Registry Split

## Goal

- Route `Markul.Arm` image publishing so `amd64` images continue to use `hub.markul.net` while `arm64` images publish to `hub-arm.markul.net`, and make both sides publish `:dev` in addition to the versioned build tag.

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Arm`
- CI file: `/home/marat/dev/git/markul/Markul.Arm/.drone.yml`
- Related project: [[agents/projects/my-infra|My Infra]]
- Related durable notes: [[personal/markul.net/markul.arm/index|markul.arm]], [[personal/tech/Infrastructure|Infrastructure]], [[personal/tech/software|Software]]

## Actions

- Checked failing `markul.auth.proxy.arm64` Drone runs on `helsinki.markul.net` and confirmed the failure happened during registry authentication against `hub.markul.net`.
- Verified that other `arm64` image pipelines were already able to push to `hub-arm.markul.net`, which isolated the problem to the `markul.auth.proxy.arm64` pipeline configuration rather than the new registry itself.
- Updated `.drone.yml` so `markul.auth.proxy.arm64` uses `repo: hub-arm.markul.net/markul.auth.proxy` and `registry: hub-arm.markul.net`.
- Committed and pushed the fix, then monitored the next Drone run until `markul.auth.proxy.arm64` successfully authenticated against `hub-arm.markul.net` instead of timing out on `hub.markul.net`.
- Checked the registries directly and confirmed the active pipelines were still publishing only versioned tags such as `2.0.0.27-linux-amd64` and `2.0.0.27-linux-arm64`, while `hub` retained older `:dev` tags and `hub-arm` had no `:dev` tags yet.
- Updated all active `get_tags` steps in `.drone.yml` to write comma-separated tags in the Drone-supported `.tags` format so each build publishes both `VERSION.BUILD-linux-ARCH` and `dev`.
- Corrected one additional config mismatch found during the pass: `markul.auth.proxy.amd64` had been left pointing at `hub-arm.markul.net` and was restored to `hub.markul.net`.
- Committed and pushed the tagging update, then monitored the next build until the split-registry publishing path was working with the new `:dev` behavior.

## Decisions

- Keep the existing versioned `VERSION.BUILD-linux-ARCH` tags because they are still used as the durable build identifiers.
- Add `dev` as an additional tag in `.tags` instead of replacing the versioned tag or re-enabling manifest publishing.
- Keep consumers architecture-aware for now: `amd64` pulls from `hub.markul.net/...:dev`, `arm64` pulls from `hub-arm.markul.net/...:dev`.
- Leave the old manifest pipelines commented out rather than deleting them so the previous multi-arch publishing shape remains visible in the repository history.

## Validation

- `ssh helsinki.markul.net 'docker events --since 2h --filter type=container ...'`
- `ssh helsinki.markul.net 'docker logs --since 10m registry-arm'`
- `ssh dev.markul.net 'docker manifest inspect hub.markul.net/markul.api:dev'`
- `ssh helsinki.markul.net 'docker manifest inspect hub-arm.markul.net/markul.api:dev'`

## Outcome

- `Markul.Arm` now publishes `amd64` images to `hub.markul.net` and `arm64` images to `hub-arm.markul.net`.
- Both sides now publish `:dev` alongside the existing versioned build tags.
- The earlier `markul.auth.proxy.arm64` registry-auth failure was resolved by correcting the registry target in `.drone.yml`.
