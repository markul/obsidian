---
note-type: session
session-date: 2026-04-27
related-project:
related-ticket:
---

# markul.web Drone Native Images

## Goal

- Update `Markul.Web` Drone image publishing to build natively per architecture and publish amd64 and arm64 images to separate registries

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Web`
- Durable note: [[personal/markul.net/markul.web/index|markul.web]]

## Actions

- Compared `Markul.Web` `.drone.yml` with the working architecture split in `/home/marat/dev/git/markul/Markul.Arm/.drone.yml`
- Removed the arm64 `buildx` step and switched it to native `plugins/docker`
- Changed the arm64 pipeline runner platform from `amd64` to `arm64`
- Split image publishing so amd64 stays on `hub.markul.net/markul.blazor` and arm64 publishes to `hub-arm.markul.net/markul.blazor`
- Disabled the multi-arch manifest pipeline by keeping it commented out as a reference block because images are now published separately by registry

## Decisions

- Follow the same native-per-architecture deployment model already used in `Markul.Arm`
- Keep architecture-specific tags with `-linux-amd64` and `-linux-arm64` suffixes while also publishing `dev`
- Keep the manifest stage inactive as commented reference because there is no longer a single registry target for both architectures

## Validation

- `git diff --check -- /home/marat/dev/git/markul/Markul.Web/.drone.yml`

## Outcome

- `Markul.Web` Drone deployment no longer uses `buildx`
- Both architectures are expected to build natively on matching Drone runners and publish to the requested registries
