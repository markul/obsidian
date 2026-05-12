---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-06
service: "[[work/alfa-bank/services/utk-mcp|utk-mcp]]"
project: "[[agents/projects/utk-mcp|UTK MCP]]"
ticket:
---

# UTK MCP Confluence Attachments

## Scope

- [[agents/projects/utk-mcp|UTK MCP]]

## Goal

- Add Confluence attachment listing and download tools to `utk-mcp`

## Actions

- Added `list_confluence_page_attachments`
- Added `download_confluence_page_attachment`
- Added Confluence attachment result models and wired them through the application interface, API tools, and Confluence client
- Reused the same attachment lookup path for both listing and download
- Added unit coverage for attachment listing, latest-version download, explicit-version download, and multi-attachment selection errors
- Verified with:
  - `dotnet test test/UtkMcp.Tests.Unit/UtkMcp.Tests.Unit.csproj --filter ConfluenceClientTests`
  - `dotnet test test/UtkMcp.Tests.Unit/UtkMcp.Tests.Unit.csproj`

## Result

- `utk-mcp` now supports Confluence page attachment discovery and version-aware download to a local temp file
- The unit suite passed `21/21`
