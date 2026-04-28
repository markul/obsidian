# markul.web

Reference: `/home/marat/dev/git/markul/Markul.Web`

## Related Notes

- [[personal/markul.net/index|markul.net]]
- [[personal/tech/software|Software]]
- [[agents/sessions/2026-04-27-markul-web-repo-review|2026-04-27 markul.web repo review]]
- [[agents/sessions/2026-04-27-markul-web-responsive-layout|2026-04-27 markul.web responsive layout]]
- [[agents/sessions/2026-04-27-markul-web-dark-theme|2026-04-27 markul.web dark theme]]
- [[agents/sessions/2026-04-27-markul-web-drone-native-images|2026-04-27 markul.web Drone native images]]

## Overview

- `markul.web` is a standalone Blazor WebAssembly frontend for the Markul home-automation platform
- The repo is focused on authenticated operational UI rather than a public-facing site: it manages users, devices, components, tasks, media views, and admin status pages
- It appears to be the web console paired with the broader backend/device stack described in [[personal/markul.net/markul.arm/index|markul.arm]]

## Main Runtime Pieces

- `Markul.Blazor` - the only project in the solution; hosts the SPA, auth wiring, pages, shared components, and typed HTTP clients
- `Pages/Devices/*` - device list, create/edit flows, component/task management, live state, charts, and image views
- `Pages/Users/*` - basic user CRUD screens
- `Pages/Admin/SystemStatus.razor` - simple environment/system-status view over known service endpoints
- `Services/*` - API/media/user clients plus custom auth-state handling for local and OIDC-backed runtime modes

## Architecture

- Single-project `.NET 6` Blazor WebAssembly solution with Bootstrap/Blazorise UI components
- Runtime configuration comes from `wwwroot/appsettings*.json`, with `ApiUrl`, `MediaUrl`, and OIDC settings for `dev-*.markul.net`
- Local development mode can bypass OIDC and inject a generated administrator token for direct API/media access
- Deployment packages the published static site into `nginx:alpine`; Drone builds and publishes multi-arch container images

## Review Notes

- `dotnet build src/Markul.Web.sln -c Release` succeeds as of `2026-04-27`
- The repo still targets `net6.0`, which is now out of support and shows `NETSDK1138` during build
- The current build emits 20 warnings, mostly nullable-reference issues in auth helpers, device pages, and image/chart components
- The current UI includes responsive layout adjustments for the left navigation, a mobile `MainLayout` height fix, and a persistent sidebar theme toggle
- Dark mode now uses a Drone-inspired palette with near-black surfaces, a deep blue sidebar, blue primary accents, and muted violet borders
- Drone image publishing now follows the `Markul.Arm` native split: amd64 publishes to `hub.markul.net`, arm64 publishes to `hub-arm.markul.net`, and the old multi-arch manifest path is kept only as a commented reference block
- Local frontend testing now expects API traffic on `http://localhost:5002/`

## Current Reading

- The codebase looks like an older but still functioning internal operations UI: the structure is small and direct compared with `markul.arm`, with most complexity pushed into the backend APIs
- Despite the `markul.web` name, this repository reads more like an authenticated admin/device-management console than a general website
