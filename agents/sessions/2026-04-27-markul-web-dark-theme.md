---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-27
service: "[[personal/services/markul.web|markul.web]]"
project:
ticket:
---

# markul.web Dark Theme

## Goal

- Add a user theme toggle to `Markul.Web` and align dark-mode colors with the Drone web app palette

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Web`
- Durable note: [[personal/services/markul.web|markul.web]]

## Actions

- Added a theme toggle button to the shared nav before the account entry in `NavMenu.razor`
- Added early theme initialization in `wwwroot/index.html` so the saved preference applies before the app renders
- Moved the shared app shell and component colors onto CSS variables in `wwwroot/css/app.css` and `wwwroot/css/markul.css`
- Updated the dark-theme variable set to use Drone-style near-black surfaces, deep blue sidebar tones, blue accents, and muted violet borders
- Adjusted nav hover and active states so the toggle and sidebar icons use theme-aware interaction colors
- Rebuilt the Blazor app to confirm the theme changes compile cleanly

## Decisions

- Keep theme state in `localStorage` and apply it from `index.html` to avoid a flash of the wrong theme during startup
- Drive dark-mode styling through shared CSS variables so future palette changes stay centralized
- Keep the Drone-inspired pass limited to shared shell and component palette changes rather than page-specific redesigns

## Validation

- `dotnet build /home/marat/dev/git/markul/Markul.Web/src/Markul.Web.sln`

## Outcome

- `Markul.Web` now has a persistent light/dark theme toggle in the sidebar
- Dark mode uses the requested Drone-inspired palette across the shared shell, cards, controls, and nav states
