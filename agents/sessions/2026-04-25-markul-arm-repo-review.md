---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-25
service: "[[personal/services/markul.arm|markul.arm]]"
project:
ticket:
---

# markul.arm Repo Review

## Goal

- Review `/home/marat/dev/git/markul/Markul.Arm` and turn that review into durable personal service notes

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Arm`
- Durable note: [[personal/services/markul.arm|markul.arm]]

## Actions

- Read the solution structure, main presentation projects, startup wiring, and representative domain/application files
- Verified the solution shape from `src/Markul.sln`
- Ran `dotnet build src/Markul.sln -c Release`
- Ran `dotnet test src/Markul.sln -c Release --no-build`
- Created the initial `markul.net` project index plus a repo description note for `markul.arm`

## Findings

- `src/Application/Markul.Application/Markul.Application.csproj:10` still references `AutoMapper` `10.0.0`, and `dotnet build` reports advisory `GHSA-rvv3-g6hj-g44x` for that package version
- `src/Tests/Markul.Tests.Shared/Markul.Tests.Shared.csproj:18` includes `Microsoft.NET.Test.Sdk`, so the shared helper assembly is treated like a test project; solution-level `dotnet test --no-build` currently produces `vstest` argument errors instead of a clean test run

## Decisions

- Keep the durable note focused on architecture, runtime roles, and current validation state rather than copying a full file inventory
- Treat `markul.arm` as the platform/backend-plus-device repo and not as an ARM-only project in the note wording

## Validation

- `dotnet build src/Markul.sln -c Release`
- `dotnet test src/Markul.sln -c Release --no-build`

## Outcome

- Added first-pass `markul.net` notes and captured two concrete follow-up items from the repository review
