---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-30
service: "[[personal/services/markul.web|markul.web]]"
project: 
ticket:
---

# 2026-04-30 markul.web .NET 10

## Goal

- Update `/home/marat/dev/git/markul/Markul.Web` from .NET 6 to .NET 10.

## Scope

- Durable note: [[personal/services/markul.web|markul.web]]
- Repository: `/home/marat/dev/git/markul/Markul.Web`
- Project: `src/Markul.Blazor/Markul.Blazor.csproj`

## Actions

- Retargeted `Markul.Blazor` from `net6.0` to `net10.0`.
- Updated Microsoft platform packages to `10.0.7`: `Microsoft.Extensions.Http`, `Microsoft.AspNetCore.Components.WebAssembly`, `Microsoft.AspNetCore.Components.WebAssembly.DevServer`, and `Microsoft.AspNetCore.Components.WebAssembly.Authentication`.
- Updated the Blazor Docker build stage from `mcr.microsoft.com/dotnet/sdk:6.0` to `mcr.microsoft.com/dotnet/sdk:10.0`.
- Preserved the existing third-party Blazorise package versions because restore/build did not require a coordinated upgrade.
- Installed Node.js `v24.15.0` LTS system-wide through NodeSource apt packages after removing the temporary user-local Node install.
- Added repo-local Playwright setup with `@playwright/test`, `playwright.config.ts`, E2E npm scripts, Chromium browser assets, desktop/mobile Chromium projects, and focused specs for the Blazor app shell, auth, navigation, dark/light theme switching, devices, capture-image server commands, users, and admin status rendering.
- Configured Playwright to start `Markul.Arm` from `src/Presentation/Markul.Console` with `dotnet run local`, then start `Markul.Web` on `http://localhost:5100`; using `localhost` is required because `index.html` selects the Blazor `Local` environment based on the hostname.
- Configured Playwright with the HTML reporter and `trace: on` so SSH-only test runs can be inspected afterward through `playwright-report/` and `test-results/*/trace.zip`.
- Set Playwright `workers: 5` and wrapped the stateful capture-image command/reload/refresh scenario in a temporary cross-worker file lock so the desktop and mobile variants run consecutively while the rest of the suite can still run in parallel.
- Updated the image refresh UI path so `DeviceImage` re-renders when loading/image state changes, the refresh spinner is centered over the image, and `DevicesClient.AddCommand` surfaces failed server-command requests.
- Added local `MediaUrl` configuration for the `Markul.Arm` media service on `http://localhost:5006/`.

## Validation

- `dotnet restore src/Markul.Web.sln`
- `dotnet build src/Markul.Web.sln --no-restore`
- `dotnet publish src/Markul.Blazor/Markul.Blazor.csproj -c Release --no-restore`
- `docker build --target publish -f src/Markul.Blazor/Docker/Dockerfile -t markul-web-dotnet10-publish-check .`
- `npm run test:e2e`

## Current State

- Restore, build, Release publish, and Docker publish-stage validation all pass on local `.NET SDK 10.0.107`.
- The build still emits 29 warnings, mostly existing nullable-reference warnings plus .NET 10 obsolescence warnings for `SignOutSessionStateManager`.
- Release publish succeeds but warns that `wasm-tools` is not installed, so Blazor publishes without size optimizations.
- Playwright is installed at the repo level and `npm run test:e2e` passes with 22 Chromium checks under 5 workers: 11 specs across desktop and mobile projects, including image click-refresh loading and timestamp assertions.
