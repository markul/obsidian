---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-13
related-project: 
related-ticket: 
---

# 2026-04-13 ufr-kpnsb-limit-service-prelive-db-connection

## Goal

- Diagnose the prelive startup failure in `ufr-kpnsb-limit-service` and fix the deployment-side cause

## Scope

- Repo: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Helm values repo: `/home/marat/dev/git/alfa/infra-helm-values`
- Related notes:
  - [[daily/2026-04-13]]

## Actions

- Read the service startup and DI code to confirm both EF Core and WorkflowCore consume `ConnectionStrings:LimitDbContext` directly
- Verified the failing stack trace comes from Npgsql connection-string parsing during WorkflowCore PostgreSQL initialization
- Compared this service's Helm `appsettings.yml` with sibling services in `infra-helm-values`
- Found that this service rendered `Password=@{PostgreSqlCredentials:Password};` without quoting, while sibling services quote the password value
- Updated `/home/marat/dev/git/alfa/infra-helm-values/ufr-kpnsb-limit-service/appsettings.yml` to render `Password='@{PostgreSqlCredentials:Password}';`

## Decisions

- Treat the incident as a deployment-template defect, not an application-code defect
- Use the same quoted-password pattern already used by sibling services to prevent reserved characters in vault passwords from breaking Npgsql parsing

## Validation

- Service code path:
  - `AddDbDataContext` uses `connectionStrings.LimitDbContext`
  - `AddWorkflowCore` uses `configuration.GetConnectionString("LimitDbContext")`
- Helm template evidence before fix:
  - `Password=@{PostgreSqlCredentials:Password};`
- Helm template evidence after fix:
  - `Password='@{PostgreSqlCredentials:Password}';`
- The reported malformed parameter `lqvhe*rorqu; pooling` is consistent with a password value corrupting the next `Pooling` token during connection-string parsing

## Follow Up

- Roll out the updated `infra-helm-values` change to prelive so the rendered appsettings file is regenerated
- If the pod is already running with the bad config, restart or redeploy it after the values change is applied
