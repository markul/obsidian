---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/utk-mcp|utk-mcp]]"
status: active
---

# utk-mcp

## Repository

- Remote: [utk-mcp](https://git.moscow.alfaintra.net/users/u_m29r2/repos/utk-mcp/)
- Local path: `/home/marat/dev/git/alfa/utk-mcp`

## Local Development

- MCP server for Alfa Jira, Bitbucket, Jenkins, Confluence, and Kibana access.
- Internal Alfa resource access should use `utk-mcp` only.

## Related Projects

- [[agents/projects/utk-mcp|UTK MCP]]

## Recent Sessions

- [[agents/sessions/2026-04-06-utk-mcp-confluence-attachments|2026-04-06 Confluence attachments]]
- [[agents/sessions/2026-04-07-utk-mcp-docs-sync|2026-04-07 docs sync]]
- [[agents/sessions/2026-04-18-utk-mcp-kibana-provider|2026-04-18 Kibana provider]]
- [[agents/sessions/2026-04-19-utk-mcp-kibana-multi-service|2026-04-19 Kibana multi-service]]

## Known Issues

- Kibana exact service filtering is intentionally based on `app_id` and `app`, not `service.name`.
