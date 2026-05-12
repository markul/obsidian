---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-07
service: "[[work/alfa-bank/services/utk-mcp|utk-mcp]]"
project: "[[agents/projects/utk-mcp|UTK MCP]]"
ticket:
---

# UTK MCP Docs Sync

## Scope

- [[agents/projects/utk-mcp|UTK MCP]]

## Goal

- Review the current `utk-mcp` repo state and align repository docs with the implemented server behavior

## Actions

- Inspected the repo structure, tool surface, Docker flow, provider clients, and test layout
- Ran `dotnet test UtkMcp.sln`
- Compared `get_jenkins_build_log`, `get_bitbucket_compare_diff`, and `download_confluence_page_attachment` to clarify the difference between inline MCP payloads and file-backed outputs
- Updated repo docs so `AGENTS.md` reflects the current implemented state instead of a future scaffold
- Translated `README.md` to Russian by request
- Moved the technical-gap list out of `README.md` and into `AGENTS.md`
- Switched `get_jenkins_build_log` and `get_bitbucket_compare_diff` to file-backed export
- Added `get_bitbucket_pull_request_diff` as a separate file-backed PR diff tool while keeping `get_bitbucket_pull_request_changes` lightweight
- Added provider download/export path config for Bitbucket and Jenkins
- Updated Docker compose to run the container as the host user and added `UID` / `GID` to the local env flow
- Forced UTF-8 decoding for JSON responses and changed JSON serialization/export settings so non-ASCII text remains readable instead of being escaped unnecessarily
- Bumped the server version to `0.2.0`
- Rebuilt the local MCP container and verified the live service reports version `0.2.0`
- Re-synced the live MCP instance after the version bump and verified both `/health` and `/` against the running container

## Decisions

- Keep `README.md` user-facing and runtime-oriented
- Keep technical gaps and contributor/agent guidance in `AGENTS.md`
- Use Russian only in explicitly requested files; keep English as the default elsewhere
- Keep changed-file listing and full PR diff retrieval as separate Bitbucket tools
- Prefer file-backed export for large diff/log payloads instead of inline MCP payloads
- Run the local container as the host user to avoid root-owned export artifacts on the bind mount
- Use readable UTF-8 JSON serialization rather than relying on escaped `\\uXXXX` output for non-ASCII text

## Validation

- `dotnet test UtkMcp.sln`
- `docker compose up --build -d`
- `curl http://127.0.0.1:1985/health`
- `curl http://127.0.0.1:1985/`

## Result

- Repo documentation now matches the implemented MCP server more closely
- `README.md` is Russian
- Technical gaps are tracked in `AGENTS.md`
- Bitbucket compare diff, Bitbucket pull request diff, and Jenkins build log large payloads are now file-backed exports
- Local downloads/export artifacts are no longer expected to be root-owned when the compose flow uses the configured `UID` / `GID`
- Live MCP health now reports version `0.2.0`, and the root endpoint confirms `streamable-http`
- Unit suite passed `23/23`
