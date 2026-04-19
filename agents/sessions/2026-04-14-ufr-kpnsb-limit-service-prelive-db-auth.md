---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-14
related-project:
related-ticket:
---

# 2026-04-14 ufr-kpnsb-limit-service-prelive-db-auth

## Goal

- Re-check the prelive PostgreSQL authentication failure in `ufr-kpnsb-limit-service` after the earlier password-quoting template fix

## Scope

- Repo: `/home/marat/dev/git/alfa/ufr-kpnsb-limit-service`
- Helm values repo: `/home/marat/dev/git/alfa/infra-helm-values`
- Related notes:
  - [[agents/sessions/2026-04-13-ufr-kpnsb-limit-service-prelive-db-connection]]
  - [[daily/2026-04-14]]

## Actions

- Re-read startup and DI wiring to confirm the service still consumes `ConnectionStrings:LimitDbContext` directly through `UseNpgsql`
- Verified the Hashicorp path loads the rendered `APPSETTINGS_FILE` and applies `@{...}` substitutions without extra password handling
- Re-checked Helm values for prelive and confirmed the rendered DB settings point to `skp-pg-pre6-vip:5433`, database `kpnsb_limits`, user `ufr_kpnsb_limit_service`, and quoted password substitution
- Compared the chart secret mapping and confirmed the DB user and password come from `$skp/ufr-kpnsb-limit-service/all/PostgreSQL_kpnsb_limit`
- Ran a local Npgsql parsing check with `Npgsql 10.0.0` and verified that `Password='пароль';` is parsed correctly, including Cyrillic and semicolon-containing samples

## Decisions

- Do not treat the current incident as a repo-side password-quoting bug; that path now looks correct
- Treat the highest-probability causes as deployment-side:
  - Vault secret differs from the actual prelive DB or PgBouncer auth entry
  - The stored password contains hidden trailing characters such as newline or carriage return
  - The VIP or auth proxy behind `skp-pg-pre6-vip:5433` is out of sync with the database user password

## Validation

- Code path:
  - `WebBuilderConfigurationExtensions` loads the rendered `APPSETTINGS_FILE` when `IS_HASHICORP=true`
  - `EnableSubstitutions("@{", "}")` resolves `ConnectionStrings:LimitDbContext`
  - `AddDbDataContext` passes `connectionStrings.LimitDbContext` into `UseNpgsql`
- Helm template:
  - `ConnectionStrings:LimitDbContext` currently renders `Password='@{PostgreSqlCredentials:Password}';`
  - `postgre-sql_user-id` and `postgre-sql_password` both come from `$skp/ufr-kpnsb-limit-service/all/PostgreSQL_kpnsb_limit`
- Local parser check:
  - `Password='пароль';` parses to password `пароль`
  - `Password='pa;ss';` parses to password `pa;ss`

## Follow Up

- From the running prelive pod, test direct login with the rendered credentials to confirm the failure reproduces outside the app
- Check the secret value for hidden trailing characters by comparing length or hex output rather than visual inspection only
- If possible, compare login through `skp-pg-pre6-vip:5433` versus the underlying DB host to rule in or out an auth proxy mismatch
