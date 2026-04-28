---
note-type: session
session-date: 2026-04-27
related-project:
related-ticket:
---

# markul.web Responsive Layout

## Goal

- Refine `Markul.Web` navigation and layout behavior for desktop and mobile local development

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Web`
- Durable note: [[personal/markul.net/markul.web/index|markul.web]]

## Actions

- Reviewed the current uncommitted UI diff and iterated on `NavMenu.razor`, `NavMenu.razor.css`, and `MainLayout.razor.css`
- Reworked the desktop sidebar so the lower navigation items live in a dedicated bottom flex group instead of hard-coded per-item offsets
- Tightened the desktop sidebar icon sizing and spacing using a shared `gap` and link-height rule
- Added mobile layout height handling in `MainLayout.razor.css` so the main content area fills the viewport under the mobile nav bar
- Rebuilt the Blazor app and verified that the served `Markul.Blazor.styles.css` on `localhost:5000` includes the updated scoped `MainLayout` rules

## Decisions

- Keep the desktop sidebar bottom cluster on a real flex layout with one spacing control instead of absolute positioning
- Keep the mobile fix in `MainLayout` rather than page-specific content CSS because the height issue belongs to the shared shell
- Preserve the current local API setting at `http://localhost:5002/` in `appsettings.Local.json`

## Validation

- `dotnet build /home/marat/dev/git/markul/Markul.Web/src/Markul.Web.sln`
- `curl http://localhost:5000/Markul.Blazor.styles.css`

## Outcome

- The desktop sidebar uses maintainable bottom-group layout rules
- The mobile main content area now fills the viewport when the updated local dev server assets are served
