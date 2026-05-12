---
tags:
  - personal/service
note-type: service
service: "[[personal/services/markul.web|markul.web]]"
---

# markul.web

Reference: `/home/marat/dev/git/markul/Markul.Web`

## Related Notes

- [[personal/tech/software|Software]]
- [[agents/sessions/2026-04-27-markul-web-repo-review|2026-04-27 markul.web repo review]]
- [[agents/sessions/2026-04-27-markul-web-responsive-layout|2026-04-27 markul.web responsive layout]]
- [[agents/sessions/2026-04-27-markul-web-dark-theme|2026-04-27 markul.web dark theme]]
- [[agents/sessions/2026-04-27-markul-web-drone-native-images|2026-04-27 markul.web Drone native images]]
- [[agents/sessions/2026-04-30-markul-web-dotnet10|2026-04-30 markul.web .NET 10]]
- [[agents/sessions/2026-05-01-markul-web-image-history|2026-05-01 markul.web image history]]
- [[agents/sessions/2026-05-02-markul-web-image-thumbnails|2026-05-02 markul.web image thumbnails]]

## Overview

- `markul.web` is a standalone Blazor WebAssembly frontend for the Markul home-automation platform
- The repo is focused on authenticated operational UI rather than a public-facing site: it manages users, devices, components, tasks, media views, and admin status pages
- It appears to be the web console paired with the broader backend/device stack described in [[personal/services/markul.arm|markul.arm]]

## Main Runtime Pieces

- `Markul.Blazor` - the only project in the solution; hosts the SPA, auth wiring, pages, shared components, and typed HTTP clients
- `Pages/Devices/*` - device list, create/edit flows, component/task management, live state, charts, and image views
- `Pages/Users/*` - basic user CRUD screens
- `Pages/Admin/SystemStatus.razor` - simple environment/system-status view over known service endpoints
- `Services/*` - API/media/user clients plus custom auth-state handling for local and OIDC-backed runtime modes

## Architecture

- Single-project `.NET 10` Blazor WebAssembly solution with Bootstrap/Blazorise UI components
- Runtime configuration comes from `wwwroot/appsettings*.json`, with `ApiUrl`, `MediaUrl`, and OIDC settings for `dev-*.markul.net`
- Local development mode can bypass OIDC and inject a generated administrator token for direct API/media access
- Deployment packages the published static site into `nginx:alpine`; Drone builds and publishes multi-arch container images

## Review Notes

- `dotnet build src/Markul.Web.sln --no-restore` succeeds as of `2026-04-30`
- `dotnet publish src/Markul.Blazor/Markul.Blazor.csproj -c Release --no-restore` succeeds as of `2026-04-30`
- Docker build validation through the `publish` stage succeeds with the `.NET 10` SDK image
- The current build emits 29 warnings, mostly nullable-reference issues plus .NET 10 `SignOutSessionStateManager` obsolescence in auth helpers
- Playwright E2E coverage exists in the repo with `@playwright/test` and desktop/mobile Chromium projects; Playwright starts `Markul.Arm` local mode with `dotnet run local` before starting the Blazor app on `http://localhost:5100` so the app selects its `Local` environment
- Current E2E coverage includes the app shell, local administrator auth, primary navigation, dark/light theme switching, seeded device list, device details/components, capture-image server-command flow with image display after reload, image click-refresh loading/timestamp behavior, device creation, users list/detail, and mocked admin system-status cards
- The capture-image E2E flow uses a cross-worker file lock so desktop and mobile project runs execute that stateful flow consecutively even when Playwright runs with multiple workers
- Playwright is configured to keep an HTML report and traces for every run; on SSH-only hosts, use `npx playwright show-report --host 0.0.0.0` and SSH port forwarding to inspect the run from a local browser
- Device details now include an `Images` tab backed by `Markul.Arm` camera history data; `Main` still shows the latest image per camera, while `Images` lists previous captures for the selected camera in a thumbnail grid backed by media files stored with a `_thumbnail.jpg` suffix. The tab keeps camera and thumbnail-column selectors plus an icon-only refresh action in a compact options card, can show one, two, or three thumbnails per row with two as the default, refreshes history for the selected camera, and loads additional thumbnails in 30-image batches when scrolling reaches the bottom. Camera history is not date-filtered.
- The current UI includes responsive layout adjustments for the left navigation, a mobile `MainLayout` height fix, and a persistent sidebar theme toggle
- Dark mode now uses a Drone-inspired palette with near-black surfaces, a deep blue sidebar, blue primary accents, and muted violet borders
- Drone image publishing now follows the `Markul.Arm` native split: amd64 publishes to `hub.markul.net`, arm64 publishes to `hub-arm.markul.net`, and the old multi-arch manifest path is kept only as a commented reference block
- Local frontend testing now expects API traffic on `http://localhost:5002/`

## Local Stack

- Start the backend/API/media/local-device stack from `Markul.Arm`:

```bash
cd /home/marat/dev/git/markul/Markul.Arm/src/Presentation/Markul.Console
dotnet run local
```

- If `dotnet watch` hits Linux inotify limits, use polling or run without watch:

```bash
DOTNET_USE_POLLING_FILE_WATCHER=1 dotnet watch run local
```

- Start the Blazor web UI from `Markul.Web`:

```bash
cd /home/marat/dev/git/markul/Markul.Web
dotnet run --project src/Markul.Blazor/Markul.Blazor.csproj --urls http://localhost:5000
```

- Expected local ports:
  - Web UI: `http://127.0.0.1:5000/`
  - API: `http://127.0.0.1:5002/`
  - Media/static images: `http://127.0.0.1:5006/`
- When accessing through VS Code Remote SSH from a Mac, forward ports `5000`, `5002`, and `5006`.
- Use `http://127.0.0.1:5000/` in the local browser if `localhost` forwarding returns `403`.
- `index.html` now treats `localhost`, `127.0.0.1`, and `::1` as the Blazor `Local` environment.
- In `Markul.Arm` local mode, generated media image links should use `http://localhost:5006/static/...`, not the HTTPS media port.

## UI Issue Reproduction

- General browser-driven checks can use Playwright from `Markul.Web` with the app already running:

```bash
cd /home/marat/dev/git/markul/Markul.Web
npx playwright test tests/e2e/devices.spec.ts -g "adds capture image" --project=desktop-chromium
```

- To inspect a layout issue during development, run an ad hoc Playwright script. Use `ignoreHTTPSErrors: true` only when the scenario still touches HTTPS localhost endpoints with an untrusted dev certificate.
- Device image capture/reload/click flow:
  - Open `http://127.0.0.1:5000/devices`.
  - Open the seeded `device-*` card.
  - Open `ServerCommands`.
  - Click `Add Command`.
  - Select `CaptureImage`.
  - Select `Camera01`.
  - Click `Add`.
  - Reload the device details page until the `Main` tab image is no longer `static/no-image-icon.png`.
  - Wait a few seconds for the command cleanup path to settle.
  - Click the image on the `Main` tab to trigger refresh/loading behavior.
- To make transient UI states reproducible, delay the relevant API response in Playwright before triggering the UI action:

```js
let delayState = false;
await page.route('http://localhost:5002/devices/1/state', async route => {
  if (delayState) {
    await new Promise(resolve => setTimeout(resolve, 2500));
  }
  await route.continue();
});

delayState = true;
await page.locator('.tab-pane.active .device-image-container img.device-image').first().click();
```

## Current Reading

- The codebase looks like an older but still functioning internal operations UI: the structure is small and direct compared with `markul.arm`, with most complexity pushed into the backend APIs
- Despite the `markul.web` name, this repository reads more like an authenticated admin/device-management console than a general website
