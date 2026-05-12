---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/utk-pwr|utk-pwr]]"
---

# utk-pwr

## Repository

- Local path: `/home/marat/dev/git/alfa/utk-pwr`

## Local Development

- Local workspace for monitoring configured objects across Bitbucket pull requests linked to `SKPDEPLOY` Jira tickets.
- Uses `utk-mcp` as the required Alfa Jira and Bitbucket integration boundary.

## Related Projects

- [[agents/projects/utk-pwr|UTK PWR]]
- [[agents/projects/utk-mcp|UTK MCP]]

## Known Issues

- `check-pwr-by-linked-tickets` is blocked by the current `utk-mcp` connector limitation for non-`SKPDEPLOY` project keys.
- Inventory has a known `skp-restructing-service` typo that should become `skp-restructuring-service`.
