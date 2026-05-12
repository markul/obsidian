---
tags:
  - personal/service
note-type: service
service: "[[personal/services/markul.arm|markul.arm]]"
---

# markul.arm

Reference: `/home/marat/dev/git/markul/Markul.Arm`

## Related Notes

- [[personal/tech/software|Software]]
- [[agents/sessions/2026-04-25-markul-arm-repo-review|2026-04-25 markul.arm repo review]]
- [[agents/sessions/2026-04-25-markul-arm-outbox-queue-refactor|2026-04-25 markul.arm outbox queue refactor]]
- [[agents/sessions/2026-04-27-markul-client-local-capture-image|2026-04-27 markul.client local capture image]]
- [[agents/sessions/2026-05-02-dev-markul-media-thumbnail-backfill|2026-05-02 dev markul.media thumbnail backfill]]

## Overview

- `markul.arm` is not just an ARM device app; it is a broader layered `.NET 8` solution for the Markul automation platform
- The repo combines central backend services, an ARM-oriented device client, shared domain/application layers, and integration infrastructure in one solution
- Primary domain focus: managed devices, components, server commands, watering tasks, media capture, and device telemetry sync

## Main Runtime Pieces

- `Markul.Api` - ASP.NET Core API for users, devices, components, and scheduled device tasks
- `Markul.Client` - ARM/device-side hosted service that can register a device, provision default components, sync state, execute commands, and expose health/metrics
- `Markul.Engine` - background processing service for scheduled and asynchronous work
- `Markul.Media` - media/image upload service
- `Markul.AuthProxy` - auth/token proxy service
- `Markul.Console` - local orchestration/process runner utility

## Architecture

- Solution layout follows `Domain -> Application -> Infrastructure -> Presentation`
- `Domain` models devices, components, users, and watering tasks
- `Application` uses MediatR-style command/query handlers and validation for business flows
- `Infrastructure` contains EF Core data access, Kafka and outbox support, auth/identity helpers, notifications, and external integrations such as Instagram and Yandex
- `Presentation` hosts the API and the long-running service entry points
- Tests exist for API, application, media, auth proxy, client, and shared helpers

## Review Notes

- `dotnet build src/Markul.sln -c Release` succeeds
- `dotnet test src/Markul.sln -c Release --no-build` currently fails at solution level with `vstest` argument errors, so the documented full-solution validation path is not healthy yet
- `AutoMapper` is pinned to `10.0.0` in the application layer and build output reports a high-severity advisory for that version

## Current Reading

- The repo looks like an actively modernized older codebase: `.NET 8` targets and new test/runtime dependencies sit alongside older solution structure and some long-lived integration modules
- The `markul.arm` name is narrower than the actual scope; this repository appears to be the main backend/device platform repo rather than only the ARM client
- As of `2026-04-26`, Drone publishing is split by architecture: `amd64` images publish to `hub.markul.net`, `arm64` images publish to `hub-arm.markul.net`, and both sides publish `:dev` alongside versioned build tags
- As of `2026-04-27`, the local `captureImage` failure was traced to `Markul.Console` overriding the `Markul.Media` HTTP pipeline in `MediaProcess`; the fix keeps the local outbox DB reset but moves it into a startup task so `/images` and the rest of the media endpoints stay mapped
- On `dev.markul.net`, `markul.media` stores files in Docker volume `markul-media-storage`, mounted at `/file-storage`; public URLs are served through `https://dev-media.markul.net/static/...`
