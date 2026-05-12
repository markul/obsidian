---
tags:
  - alfa-bank/service
note-type: service
service: "[[work/alfa-bank/services/utk-mcp|utk-mcp]]"
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

## Known Issues

- Kibana exact service filtering is intentionally based on `app_id` and `app`, not `service.name`.
- Kibana aggregation is available through `aggregate_kibana_logs`; pass real Elasticsearch field names such as `app_id.keyword` and `log.logger.keyword`.
