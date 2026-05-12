---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-10
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-10 UTK AI Chat Log Tool Policy

## Goal

- Tune `utk-ai` Chat log-search behavior around MCP `kibanaUrl` evidence, backend pre-search removal, aggregate-vs-raw tool choice, and latest-count date filtering.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`

## Actions

- Compared the known-good Kibana Discover URL shape against the current `utk-ai`/`utk-mcp` flow and kept `utk-ai` consuming MCP-provided `kibanaUrl` values rather than locally rebuilding links.
- Removed backend pre-search from Chat log context: `ChatLogContextBuilder` now provides planning context and resolved service hints, while actual Kibana evidence must come from native agent tool results.
- Updated log-skill/tool guidance so the model prefers `search_aggregated_logs` for normal investigations, latest/top errors, repeated failures, patterns, and summaries; `search_logs` is reserved for exact document/request/trace/Lucene/raw-row searches.
- Changed raw and aggregated agent tool inputs plus MCP request construction so omitted `dateFromUtc`/`dateToUtc` remain absent from `utk-mcp` calls.
- Added prompt, skill, and tool-catalog guidance that "last N" or "latest N" log/error requests should omit date filters unless the user explicitly provides a date or time window.
- Added a latest-count prompt heuristic so planning context suppresses the default UTC window hint for prompts such as "последние 100 ошибок" unless the prompt also names a date or time window.
- Removed internal assistant transcript marker paths:
  - Application no longer emits visible `[[UTK_TOOL_INTENT]]` chunks.
  - The system prompt no longer asks the model to wrap error text in `[[UTK_ERROR_QUOTE]]`.
  - The Web renderer no longer parses `.chat-tool-intent` or `.chat-error-quote`; assistant content renders as plain sanitized Markdown.
  - Tool intent stays in debug/tool-call events.
- Removed follow-up legacy cleanup after the marker removal:
  - Deleted unused backend pre-search downstream-candidate extraction.
  - Removed the test-only `AppendLogSearchContext` formatter branch and obsolete verdict/total helper tests.
  - Removed the unused `AgentToolCall.Intent` field and intent debug branch.
  - Renamed the remaining visible-response cleanup helper to `ChatVisibleResponseSanitizer`.
- Fixed log follow-up context:
  - Log turns now include recent conversation history instead of replacing it with only planning context.
  - Recent MCP/tool debug evidence is summarized into the model request so follow-ups can reuse exact prior fields such as `sourceContext=CronJob.CronJobExecutors.CronJobExecutor` and existing `kibanaUrl` examples.
  - Prompt policy now requires every displayed concrete error/group/pattern to have a real `kibanaUrl` example from current or recent tool evidence.
  - Narrowed zero-result follow-ups are instructed not to fall back to unrelated broader raw logs as if they matched.
- Increased Chat native tool-call iteration limit from 3 to 8 for deeper multi-step diagnostics without letting weak search loops run too long.
- After analyzing conversation `5249da25-f74c-4356-be28-9e84db4d434a`, clarified the model-facing tool policy instead of forcing tool rewrites in code:
  - `search_aggregated_logs` now appears before raw `search_logs` in tool metadata.
  - The system prompt, skill rules, and planning context explicitly say that "last N" and "latest N" errors/logs are aggregate investigation requests.
  - Raw `search_logs` is described as exact raw lookup only for document IDs, trace/request IDs, exact Lucene queries, or explicit raw-row requests.
- Fixed a Chrome hard-reload composer edge case where the browser could restore textarea text while Blazor still had an empty `DraftMessage`, leaving the send button disabled. Chat now reads the first-render DOM value back into `ChatSessionState`, and the Playwright chat spec simulates this restored-draft state.
- Improved chat Kibana evidence link formatting:
  - The prompt now asks for compact Markdown links like `[Open in Kibana - 07:10:05 UTC](<kibanaUrl>)`.
  - `ChatVisibleResponseSanitizer` rewrites brittle inline Kibana Discover links into angle-bracket Markdown destinations so Discover URLs with parentheses render as links instead of leaking raw Markdown.
  - The Web chat renderer applies the same normalization for already-persisted assistant messages.
  - Timestamp-only link labels are normalized to `Open in Kibana - <timestamp>`.
- Added native chat tools for Alfa internal systems, all backed by `utk-mcp`:
  - `query_jira_tickets` maps exact `issueKey` reads and JQL search to Jira MCP tools, with optional development metadata and remote links.
  - `query_confluence_pages` maps `pageId` reads and CQL search to Confluence MCP tools.
  - `query_jenkins_builds` maps service build history, job reads, exact build reads, and optional build log export to Jenkins MCP tools.
  - Prompt policy now requires Jira, Confluence, and Jenkins answers to use these tools instead of memory.
  - `snxctl status` was connected before inspecting/using Alfa internal MCP capability.
- Wired the `internal-systems` skill into the active chat selector and planning-context path:
  - It triggers for Jira issue keys, Jira/JQL wording, Confluence/CQL/page wording, Jenkins/job/build/console wording, and short follow-ups after recent internal-system context.
  - The selected-skill debug entry now shows `internal-systems` with allowed tools and evidence rules.
  - Planning context routes Jira to `query_jira_tickets`, Confluence to `query_confluence_pages`, and Jenkins to `query_jenkins_builds`.
  - Recent tool evidence context was generalized from Kibana-specific wording to MCP evidence so follow-ups can reuse ticket keys, page IDs, job paths, and build numbers.
- Flattened generic tool presentation:
  - Available tools now share the same `Tool` category in the tool catalog.
  - The Web available-tools modal no longer displays a category row.
  - The system prompt says all Alfa integration tools, including Kibana and Jira/Confluence/Jenkins, are backed by `utk-mcp`, and generic capability answers should not group tools by backend/source/internal status.
  - After the model still split "what tools are available?" into Kibana and Alfa integration sections, the prompt now explicitly requires one flat list/table for generic tool-list answers and forbids sections such as logs, Kibana, integrations, Jira, Confluence, Jenkins, internal systems, or `utk-mcp`.
  - The prompt also says skills are internal routing/planning context and should not be mentioned in generic tool-list answers unless the user explicitly asks about skills.
  - Domain-specific distinctions now belong to selected skills and their planning context.
- Added prompt guidance that UTK AI may use emoji when appropriate and helpful for readability, while keeping technical identifiers unchanged.
- Added a preferred Russian response pattern for "who are you / what can you do" prompts, including UTK AI identity, log/Jira/Confluence/Jenkins capabilities, example requests, and the closing "Чем могу помочь? 🚀".
- Improved Confluence service-page lookup:
  - `query_confluence_pages` now accepts `serviceName` for main service description pages.
  - The tool builds a CQL search around the `Service :: <serviceName>` pattern.
  - Prompt and planning context tell the model to display full Confluence links, prefixing relative `WebUrl` values with `https://confluence.moscow.alfaintra.net`, matching URLs such as `https://confluence.moscow.alfaintra.net/spaces/LOANMGR/pages/3195430948/Service+skp-product-change-workflow-service`.
- Added stricter Confluence documentation handling after reviewing the latest chat conversation:
  - Selector now treats documentation wording such as `документация`, `документац`, `documentation`, `docs`, and `титульные страницы` as `internal-systems` requests.
  - Added `query_confluence_service_pages` for batch main service page checks. It reads the LOANMGR KPNSB services index page `808605038` first, then searches missing services by `Service :: <serviceName>`.
  - The chat tool loop now retries once with a hard tool-use instruction when an evidence-backed selected skill produces an answer without the required evidence tool; a `list_services` call alone is not enough for Confluence documentation claims. If the retry still has no required evidence tool call, it blocks the unsupported answer instead of displaying fabricated status tables.
- Improved Confluence service-page lookup after the index body proved to contain only a dynamic children macro:
  - `utk-mcp` now exposes `list_confluence_child_pages` for child or descendant Confluence pages under an index page.
  - `query_confluence_service_pages` queries descendants of LOANMGR index page `808605038` before falling back to search.
  - Single-service and batch service-page lookup now prefer exact-title CQL `title = "Service :: <serviceName>"` before fuzzy `title ~` or text search.
  - Batch lookup still accepts only exact main page titles, so worker/algorithm/CNT/method pages are not treated as service description pages.
- After checking conversation `b5d3cef4-cd1e-4861-a7e8-e5970ad83bf0`, fixed the batch Confluence path so a Confluence `500` from `list_confluence_child_pages` is recorded as an MCP warning and does not abort the answer; `query_confluence_service_pages` now continues to exact-title CQL fallback.
- Refined the batch path to query direct index children first, then descendants only for still-missing services, then exact-title/fuzzy search. Live Confluence returned `33` direct children for index page `808605038`, while the descendant endpoint still returned `500`.
- Added Bitbucket support to the Chat internal-systems flow:
  - New same-level native chat tool `query_bitbucket_repositories`.
  - Routes to `utk-mcp` Bitbucket operations for repository details, branch metadata, pull request listing, pull request changed files, pull request diff export, and compare diff export.
  - Accepts `projectKey`/`repoSlug` or a full `repositoryUrl`; personal repository URLs such as `/users/u_m29r2/repos/utk-mcp` map to project key `~u_m29r2`.
  - Internal-systems prompt, skill rules, tool catalog, required-evidence guard, and "who are you" capability pattern now include Bitbucket.
  - Required-evidence guard now resolves the current user request text instead of scanning the whole system prompt, so Bitbucket evidence is not rejected just because the system prompt also mentions Confluence/Jenkins.
  - Bitbucket MCP lookup failures are returned as tool evidence for the model to report instead of aborting the whole chat turn.
- After reviewing conversation `215825f4-47f0-4605-8f1c-0d912e9440ed`, fixed two Bitbucket follow-up gaps:
  - `repositoryUrl` parsing now supports clone URLs such as `https://git.moscow.alfaintra.net/scm/ufrkpnsb/skp-product-change-workflow-service.git`.
  - Prompt and tool-result guidance temporarily described repository file content as unsupported and blocked cloning/reading internal repositories outside `utk-mcp`.
- Added Bitbucket repository tree and file-content support:
  - `utk-mcp` now exposes `get_bitbucket_repository_tree` for root/path browsing and `get_bitbucket_file_content` for file reads through the Bitbucket browse API, with optional `at` refs.
  - `query_bitbucket_repositories` now routes `browsePath=true` to tree browsing and `path` plus `includeContent=true` to file reads; `branchName` can still be used as the optional ref for file/tree requests.
  - Chat prompt, skill rules, and tool catalog now describe repository structure and file reads as supported MCP-backed evidence instead of telling the model to report a limitation.
- Added Bitbucket file blame support:
  - `utk-mcp` now exposes `get_bitbucket_file_blame`, returning paged blame spans with line number, span length, commit id/display id, author, author email, timestamp, and file name.
  - `query_bitbucket_repositories` now routes `path` plus `includeBlame=true` to `get_bitbucket_file_blame`; `blameStart` controls pagination start and `limit` controls span/page size.
  - Live validation showed the deployed Bitbucket Server returns blame data with `blame=true` and not with `blame=true&noContent=true`, so the MCP client requests blame with content included and maps only the blame spans.
- Refined Bitbucket blame output:
  - `get_bitbucket_file_blame` now merges Bitbucket `lines` with blame spans into per-line rows so UTK AI can render a compact GitHub-style blame view without separately reading file content.
  - Merged rows include line number, source text, commit id/display id, author, email, timestamp, and file name.
  - The same result now also includes top-level `lineCount` and `content`, derived from the returned page lines, so file-content and blame evidence arrive in one MCP response.
  - Prompt/tool-result guidance now asks UTK AI to display blame as commit/author/date/line/source rows and mention `nextPageStart` when more rows exist.
  - Live MCP validation on `2026-05-11` against `UFRKPNSB/ufr-kpnsb-limit-service` `README.md` on `master` confirmed the result includes both `spans` and merged `lines` with source text plus commit/author/date metadata.
  - Follow-up live MCP validation confirmed the combined result includes `content`, `lineCount`, merged `lines`, and pagination metadata.
- Refined Bitbucket blame output toward the GitHub blame page shape:
  - `get_bitbucket_file_blame` now returns `groups` that combine contiguous source lines by blame span with line range, content, commit id/display id, commit message, author, email, timestamp, and the included line rows.
  - When Bitbucket browse blame omits commit messages, `utk-mcp` enriches the current page by reading unique commit details and applying the message to spans, groups, and lines.
  - `utk-ai` prompt/tool guidance now prefers `groups` for blame answers so the assistant can show commit metadata once beside grouped source lines, while keeping `content` available for plain file-content requests.
- Changed chat Markdown rendering so displayed assistant links open in new browser tabs with `target="_blank"` and `rel="noopener noreferrer"`.
- Kept explicit windows intact for monitoring and plain Kibana investigations; nullable result dates now format as `<none>` where a tool result has no explicit window.
- Restarted the API and Web in canonical tmux panes after validation.

## Validation

- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 28 tests after adding follow-up context and generic-24h coverage.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 30 tests after clarifying the aggregate-first prompt/tool policy.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 32 tests after Kibana link normalization.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 36 tests after adding Jira/Confluence/Jenkins agent tool catalog and prompt coverage.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 41 tests after wiring the `internal-systems` skill selector and planning context.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 43 tests after flattening generic tool presentation.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 44 tests after adding emoji prompt coverage.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 45 tests after adding the "who are you" response-pattern prompt coverage.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 48 tests after adding Confluence service-page lookup prompt/catalog/selector coverage.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 56 tests after adding batch Confluence service-page lookup and the required-evidence-tool guard.
- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 56 tests after exact-title Confluence lookup and descendant-index support.
- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed after exact-title Confluence lookup and descendant-index support.
- `dotnet test UtkMcp.sln --no-restore --disable-build-servers` passed with 37 tests after adding Bitbucket tree/file tools.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 61 tests after wiring Bitbucket file/tree prompt and tool routing.
- `dotnet build UtkMcp.sln --no-restore --disable-build-servers` passed after adding Bitbucket tree/file tools.
- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed after wiring Bitbucket file/tree prompt and tool routing.
- `dotnet test UtkMcp.sln --no-restore --disable-build-servers` passed with 38 tests after adding Bitbucket blame.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 61 tests after adding Bitbucket blame routing and prompt/tool metadata.
- `dotnet build UtkMcp.sln --no-restore --disable-build-servers` passed after adding Bitbucket blame.
- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed after adding Bitbucket blame routing.
- `dotnet test UtkMcp.sln --no-restore --disable-build-servers` passed with 38 tests after adding merged blame lines.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 61 tests after adding GitHub-style blame prompt guidance.
- `dotnet build UtkMcp.sln --no-restore --disable-build-servers` passed after adding merged blame lines.
- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed after adding GitHub-style blame prompt guidance.
- `dotnet test UtkMcp.sln --no-restore --disable-build-servers` passed with 38 tests after adding grouped blame output and commit-message enrichment.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 61 tests after updating blame display guidance to prefer groups.
- `dotnet build UtkMcp.sln --no-restore --disable-build-servers` passed after adding grouped blame output and commit-message enrichment.
- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed after updating blame display guidance to prefer groups.
- `dotnet test Utk.Ai.slnx --no-restore --disable-build-servers` passed with 56 tests after making descendant lookup failure fall back to exact-title CQL.
- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed after making descendant lookup failure fall back to exact-title CQL.
- `dotnet test UtkMcp.sln --no-restore` passed with 35 tests after adding `list_confluence_child_pages`.
- `dotnet build UtkMcp.sln --no-restore` passed after adding `list_confluence_child_pages`.
- `npm run test:e2e -- --project=chromium e2e/chat.spec.ts` passed with the restored-draft regression.
- `git diff --check` passed.
- `curl -sS -o /dev/null -w '%{http_code}' http://localhost:9595/` returned `200`.
- `curl -sS -o /dev/null -w '%{http_code}' http://localhost:9596/health` returned `200`.
- `curl -fsS http://127.0.0.1:1985/health` returned `{"status":"ok","name":"utk-mcp","version":"0.4.0"}` after Docker rebuild.
- Browser chat smoke for `найди ufr-kpnsb-product-request-workflow-service документацию` returned the exact Confluence page `https://confluence.moscow.alfaintra.net/spaces/LOANMGR/pages/1204718696/Service+ufr-kpnsb-product-request-workflow-service`; debug trace showed `list_confluence_child_pages` warning followed by exact CQL `title = "Service :: ufr-kpnsb-product-request-workflow-service"`.
- Browser chat smoke for a two-service title-page request returned Markdown links for both `ufr-kpnsb-product-request-workflow-service` and `skp-product-change-workflow-service`; debug trace showed direct child lookup found the first page, descendant lookup failed with Confluence `500`, and exact CQL found the second page.
- Browser chat smoke for `покажи bitbucket репозиторий https://git.moscow.alfaintra.net/users/u_m29r2/repos/utk-mcp` returned repository details and full links; debug trace showed `query_bitbucket_repositories` calling `get_bitbucket_repository` with `projectKey=~u_m29r2` and `repoSlug=utk-mcp`.
- Browser chat smoke for clone URL `https://git.moscow.alfaintra.net/scm/ufrkpnsb/skp-product-change-workflow-service.git` returned repository details; debug trace confirmed `projectKey=UFRKPNSB` and `repoSlug=skp-product-change-workflow-service`.
- MCP smoke confirmed `tools/list` advertises `get_bitbucket_repository_tree` and `get_bitbucket_file_content`.
- MCP smoke for `get_bitbucket_repository_tree` against `~U_M29R2/utk-mcp` returned root entries including `.dockerignore`, `.env.example`, `.gitignore`, `.kilocode`, and `AGENTS.md`.
- MCP smoke for `get_bitbucket_file_content` against `~U_M29R2/utk-mcp` `AGENTS.md` with `maxLines=3` returned `# AGENTS.md`, `lineCount=3`, and `truncated=true`.
- MCP smoke confirmed `tools/list` advertises `get_bitbucket_file_blame`.
- MCP smoke for `get_bitbucket_file_blame` against `UFRKPNSB/ufr-kpnsb-limit-service` `README.md` on `master` returned `size=5`, `nextPageStart=5`, and blame spans with commit ids, author names, email addresses, timestamps, and line ranges.
- MCP smoke for `get_bitbucket_file_blame` against `UFRKPNSB/ufr-kpnsb-limit-service` `README.md` on `master` returned top-level `content`, grouped blame entries, populated `commitMessage` values such as `UTK3-200 Очистка проекта`, merged line rows, `nextPageStart=5`, and `isLastPage=false`.
- Local stack check:
  - tmux session `utk-ai` has Web pane 0, API pane 1, and Skaffold deps pane 2.
  - `GET http://127.0.0.1:9596/health` returned `Healthy`.
  - `GET http://127.0.0.1:9595/` returned HTML.
  - PostgreSQL on `127.0.0.1:15432` accepted `select 1` with the configured `postgres/postgres` credentials.

## Current State

- Chat native tool calls remain WIP in the local dirty tree.
- For log questions, the backend now selects the skill and provides planning context only; the model must call an available Kibana tool before making claims about matching or missing logs.
- `search_aggregated_logs` is the default preferred tool for broad/latest/top log investigations.
- For exact lookups such as document ID, trace ID, request ID, exact Lucene query, or user-requested raw rows, `search_logs` remains available.
- Tool choice is still model-driven; broad `search_logs` calls are not forcibly rewritten in the backend.
- For "last N" or "latest N" log requests, the tool layer no longer adds a default last-24-hours filter when dates are omitted.
- For main Confluence service description pages, the model should call `query_confluence_pages` with `serviceName`; the adapter searches for `Service :: <serviceName>` and the answer should show a full Confluence URL.
- For main Confluence service description pages, `utk-ai` now tries exact-title CQL `title = "Service :: <serviceName>"` before fuzzy `title ~` search.
- For many-service Confluence title-page checks, the model should call `query_confluence_service_pages` with `serviceNames`; the adapter checks LOANMGR index descendants through `utk-mcp` before exact-title and fuzzy fallback search, and unsupported per-service tables without tool calls are blocked by the tool loop guard.
- For Bitbucket repository structure, file-content, or blame requests, the model should call `query_bitbucket_repositories` with `browsePath=true` for tree browsing, `path` plus `includeContent=true` for file reads, or `path` plus `includeBlame=true` for GitHub-style merged blame; repository file access stays inside `utk-mcp`.
- Bitbucket blame evidence is now combined and grouped: use `groups` for GitHub-like blame presentation, `lines` for exact row-level metadata, and top-level `content` when the user also wants the file text.

## Decisions

- Do not restore backend pre-search for Chat log questions; keep evidence acquisition model-driven through native tool calls.
- Keep `utk-mcp` responsible for outbound Kibana Discover URL generation.
- Treat date filters in chat log tools as optional arguments; latest-count requests should be count/order based unless the user explicitly names a window.

## Follow-Up

- Browser-smoke a real Chat request for "last N errors" and confirm the debug MCP request omits `dateFromUtc` and `dateToUtc`.
- Browser-smoke an explicit-window Chat request and confirm the debug MCP request still includes the requested UTC date range.
