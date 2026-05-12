---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-27
service: "[[personal/services/markul.web|markul.web]]"
project:
ticket:
---

# markul.web Repo Review

## Goal

- Review `/home/marat/dev/git/markul/Markul.Web` and turn that review into durable personal service notes

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Web`
- Durable note: [[personal/services/markul.web|markul.web]]

## Actions

- Read the vault policy, the existing `markul.net` notes, and the current placeholder `markul.web` note
- Reviewed the solution, project file, startup wiring, representative pages, service clients, config files, Docker packaging, and Drone pipelines
- Ran `dotnet build /home/marat/dev/git/markul/Markul.Web/src/Markul.Web.sln -c Release`
- Replaced the placeholder `markul.web` note with a first-pass repo description and updated the parent `markul.net` index

## Findings

- `src/Markul.Blazor/Markul.Blazor.csproj:4` targets `net6.0`, and the current SDK reports `NETSDK1138` because that framework is out of support
- `dotnet build` succeeds but emits 20 warnings, mostly nullable-reference issues in `Services/CustomAuthenticationStateProvider.cs`, `Pages/Devices/View.razor`, and related UI components
- The repo is a narrow frontend-only solution with a single Blazor WebAssembly project, not a broad platform repository like `markul.arm`

## Decisions

- Describe `markul.web` as an authenticated admin/device-management console rather than a generic website
- Keep the durable note focused on runtime role, deployment shape, and current maintenance signals instead of duplicating a file inventory

## Validation

- `dotnet build /home/marat/dev/git/markul/Markul.Web/src/Markul.Web.sln -c Release`

## Outcome

- Added a durable repo description for `markul.web` and updated the parent `markul.net` index to reference the completed review
