---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-27
related-project:
related-ticket:
---

# markul.client Local Capture Image

## Goal

- Fix the local-mode `captureImage` flow in `Markul.Client` after uploads started failing with `HttpRequestException` and HTTP `404` from the media upload request.

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Arm`
- Local console media runner: `/home/marat/dev/git/markul/Markul.Arm/src/Presentation/Markul.Console/Processes/MediaProcess.cs`
- Related durable notes: [[personal/markul.net/markul.arm/index|markul.arm]]

## Actions

- Traced the failing stack from `DeviceManager.ProcessDirectives` to `MediaClient.SendImage` and confirmed the upload endpoint is `POST /images`.
- Reproduced the failure under `dotnet run local` and confirmed the `captureImage` upload still failed with HTTP `404` even though the client was targeting the local media host.
- Probed `http://localhost:5006` directly and found `/images`, `/status`, and `/metrics` all returned `404`, which showed the problem was in the local media host rather than in `Markul.Client`.
- Traced that behavior to `src/Presentation/Markul.Console/Processes/MediaProcess.cs`, where the local runner used `builder.Configure(...)`; in this hosting path that replaced the normal `Startup.Configure` pipeline, leaving the media host with no mapped endpoints.
- Replaced that local-only `builder.Configure(...)` block with a startup task that resets the local outbox database before the host starts serving requests, preserving the normal media pipeline.
- Reverted the earlier exploratory client/startup and HTTP upload changes after the live reproduction showed they were not needed for the local failure.
- Reran `dotnet run local`, resent `POST /devices/1/server-command` for `captureImage`, and confirmed the client uploaded successfully and emitted an `ImageReceived` event consumed by `Markul.Engine`.

## Decisions

- Keep the local media database reset in a startup task instead of `builder.Configure(...)` so the console runner cannot accidentally replace the HTTP pipeline for that service.
- Keep the final patch minimal: only `MediaProcess` changes were required for the reproduced local `captureImage` failure.

## Validation

- `dotnet run local`
- `curl 'http://localhost:5002/devices/1/server-command' -H 'authorization: LocalToken {"UserId":1,"Role":"Administrator"}' -H 'content-type: application/json; charset=utf-8' --data-raw '{"componentId":4,"serverCommand":3}'`
- `dotnet build src/Presentation/Markul.Console/Markul.Console.csproj -c Debug --no-restore`

## Outcome

- The local console media host now preserves its real ASP.NET Core pipeline while still resetting the local outbox database before startup.
- The reproduced `captureImage` flow now succeeds under `dotnet run local`: the client logs `Sent image: https://localhost:5007/static/images/...jpg` and the local engine handles the emitted `ImageReceived` event.
- The final code change for this issue is limited to `src/Presentation/Markul.Console/Processes/MediaProcess.cs`.
