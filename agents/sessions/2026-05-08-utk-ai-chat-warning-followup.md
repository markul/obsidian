---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-08
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-08 UTK AI Chat Warning Followup

## Goal

- Fix chat log-search follow-ups so warning-level requests are evidence-backed by Kibana/MCP instead of inferred by AlfaGen.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related conversation: `53f54416-c0ed-4902-a425-6eed135e1ac2`

## Actions

- Investigated conversation `53f54416-c0ed-4902-a425-6eed135e1ac2`; the first `анализ ошибок...` turn searched Kibana for `Error`, but the follow-up `а предупреждения ?` did not run MCP and AlfaGen answered without evidence.
- Extended the backend `log-search` skill trigger to recognize warning terms and Russian stems such as `ошиб` and `предупрежд`, covering forms like `ошибок` and `предупреждения`.
- Added severity resolution to the direct chat log-search path so the MCP request can use `Warning`, `Debug`, `Information`, or default `Error`.
- Updated log-search evidence prompt wording to refer to matching logs rather than only errors.
- Investigated latest conversation `1c720a18-fb63-4bbf-bf4e-88e043644ab0`; explicit request `проверь логи skp-email-service` incorrectly searched the previous configured service because chat treated `Monitoring:Targets` as a hard allowlist.
- Changed chat log target resolution so explicitly typed hyphenated service names create transient Kibana search targets, defaulting to `dev` unless the prompt specifies another environment.
- Updated `search_logs` tool execution and agent skill/tool metadata so arbitrary explicit service names are allowed; configured targets are now suggestions, not the only searchable services.
- Changed downstream log checks so the backend `log-search` skill only performs the primary service search; downstream service candidates are shown to AlfaGen, and any downstream search must be initiated by the model through a `search_logs` tool call.
- Expanded backend skill selection so prompts such as `проверь ufr-kpnsb-product-request-workflow-service` select `log-search` directly instead of depending on model-generated tool-call JSON.
- Added chat composer input history in the Web client: the last 10 submitted prompts are persisted in browser `localStorage`, Up/Down browse them, and Down past the newest restores the in-progress draft.
- Added an application-level `IAgentToolExecutor` with an infrastructure `AgentToolExecutor`; chat model tool calls and backend-selected `log-search` MCP calls now execute through this single executor path, while `ChatHub` only parses/orchestrates and forwards debug entries to SignalR.
- Replaced the one-shot chat tool handler with a bounded agent loop that lets AlfaGen request up to three tool calls before the final answer; added a correction step for cases where AlfaGen announces a log-check action in prose instead of emitting strict tool-call JSON.
- Split chat log-search output into visible phases: Phase 1 primary-service analysis is displayed before downstream work; model downstream intent is displayed; downstream searches run through `AgentToolExecutor`; the downstream phase must point back to the original primary-service error and include an error-flow visualization.
- Removed internal phase labels from user-facing chat log-search answers; the backend still sequences primary and downstream work, but prompts now ask AlfaGen to answer in natural sections and `ChatHub` strips accidental labels such as `Phase 1` or `Primary Service`.
- Highlighted visible model intent in chat answers: natural-language downstream intent is rendered as a Markdown quote, and direct model `search_logs` tool calls now insert a short highlighted intent sentence before the MCP search runs.
- Added persisted chat prompt history for AlfaGen calls: every `ChatHub` model request now writes a `chat_prompt_history` row linked to the assistant `chat_messages.id`, with per-message sequence, purpose, model name, request messages JSON, response content, chunk counts, and error state.
- Fixed a chat-history EF tracking conflict by making `EnsureConversationAsync` reuse a locally tracked conversation before querying or adding a new one.
- Investigated conversation `5a6e4da7-77a4-433d-a0b5-f233e170f8b6`; AlfaGen saw downstream candidate `skp-email-service` but answered `Имеет смысл проверить его логи` without initiating a downstream tool call, leaving only one prompt-history row.
- Expanded natural-language downstream intent detection for phrases such as `имеет смысл проверить`, `стоит проверить`, `нужно проверить`, `should check`, and `worth checking`; tightened prompt wording so AlfaGen states intent rather than merely suggesting a check.
- Replaced the downstream prose-intent fallback with a strict JSON tool-call contract: model tool calls must include top-level `tool`, `intent`, and `arguments`; `ChatHub` displays `intent` as the highlighted callout and then executes the tool.
- Updated tool-call parsing so mixed model output with a primary summary plus fenced JSON keeps the non-JSON primary summary visible, strips the JSON from the chat transcript, displays the JSON `intent`, and then runs the tool.
- Fixed over-highlighting in chat rendering: tool intent now uses internal `[[UTK_TOOL_INTENT]]` markers rendered by the Web as `.chat-tool-intent`, while ordinary Markdown blockquotes return to muted quote styling.
- Removed the client-side error quote classifier; error quote highlighting now comes from model-provided `[[UTK_ERROR_QUOTE]]...[[/UTK_ERROR_QUOTE]]` markers, which the Web renders as red `.chat-error-quote` blocks.
- Changed debug console entries from always-expanded articles to collapsed-by-default `<details>` disclosures, keeping direction/timestamp visible in each summary.
- Tightened collapsed debug console rows so each summary stays compact instead of stretching vertically; expanded entries add spacing only around their content.
- Fixed mixed prose-plus-tool responses structurally: `ChatHub` now parses fenced/balanced JSON tool-call candidates instead of first-brace/last-brace spans, strips only the exact structured tool JSON block from visible output, and does not remove surrounding natural-language prose.
- Added persisted chat debug console history: raw `ChatDebug` events are stored in `chat_debug_entries` with conversation/message ids, sequence, direction, content, append flag, and timestamp; `GetHistory` now returns debug entries so the Web client can replay the same console output when the conversation is reloaded.

## Validation

- `dotnet build Utk.Ai.slnx` passed.
- Restarted API and Web on `http://localhost:9596` and `http://localhost:9595`.
- Browser validation sent `анализ ошибок в skp-product-change-workflow-service за 24 часа`, then `а предупреждения ?`; debug output showed `severity: Error` for the first MCP search and `severity: Warning` for the follow-up. Screenshot: `tmp/utk-ai-chat-warning-followup.png`.
- Browser validation sent `проверь логи skp-email-service` in a new chat; debug output showed `services: skp-email-service`, did not include the previous `ufr-kpnsb-product-request-workflow-service`, and the persisted response found `3,404` errors. Conversation: `84dea7c2-2a26-4608-b2e8-ba3f1d1efd0a`. Screenshot: `tmp/utk-ai-any-service-log-search.png`.
- Browser validation sent `проверь ufr-kpnsb-product-request-workflow-service`; debug output showed the backend-selected `log-search` skill, the primary MCP search for `ufr-kpnsb-product-request-workflow-service`, a model `TOOL REQUEST`, and then the downstream MCP search for `skp-email-service`. Conversation: `674b6914-bc88-4939-b92e-71c10921731c`. Screenshot: `tmp/utk-ai-downstream-log-check-service-only.png`.
- Browser validation seeded composer history and confirmed ArrowUp returned `third stored prompt`, second ArrowUp returned `second stored prompt`, ArrowDown returned `third stored prompt`, and second ArrowDown restored `draft text`. Screenshot: `tmp/utk-ai-chat-input-history.png`.
- Browser validation sent `проверь логи skp-email-service` and confirmed debug output still included `skill: log-search`, `utk-mcp tool call: search_logs`, `services: skp-email-service`, and `utk-mcp tool result: search_logs` through the executor path. Screenshot: `tmp/utk-ai-agent-tool-executor-smoke.png`.
- Browser validation sent `проверь логи ufr-kpnsb-product-request-workflow-service` and confirmed the revised flow: primary service MCP search first, downstream candidate instruction present, AlfaGen model tool request after primary evidence, downstream `services: skp-email-service` MCP request after that tool request, and a final answer that included downstream evidence. Screenshot: `tmp/utk-ai-llm-downstream-tool-request.png`.
- Browser validation confirmed phased output for `проверь логи ufr-kpnsb-product-request-workflow-service`: visible Phase 1 primary analysis, visible model intent to check `skp-email-service`, downstream MCP request after that intent, visible Phase 2 downstream result with `3,404` hits, and an error-flow diagram tying `skp-email-service` back to the original `POST /Email/v10/send` HTTP 500 failure. Screenshot: `tmp/utk-ai-phased-downstream-flow.png`.
- Browser validation sent `проверь логи ufr-kpnsb-product-request-workflow-service` after removing visible phase labels; confirmed the answer contained natural log-analysis text, the service name, and an error-flow diagram, with no `Phase 1`, `Primary Service`, or `Downstream:` labels. Screenshot: `tmp/utk-ai-natural-log-output.png`.
- Browser validation sent `проверь логи ufr-kpnsb-product-request-workflow-service` and confirmed the model intent `Проверю логи skp-email-service...` rendered as a styled quote with blue left border/background, while surrounding primary analysis remained normal text and no phase labels appeared. Screenshot: `tmp/utk-ai-highlighted-model-intent.png`.
- `dotnet ef migrations add AddChatPromptHistory --project src/Utk.Ai.Infrastructure/Utk.Ai.Infrastructure.csproj --startup-project src/Utk.Ai.Api/Utk.Ai.Api.csproj --output-dir Persistence/Migrations` created migration `20260508094132_AddChatPromptHistory`.
- Browser validation sent `hi prompt history smoke`; PostgreSQL confirmed one `chat_prompt_history` row linked to the assistant message with `purpose=initial-response`, `model_name=MiniMaxAI/MiniMax`, request JSON, and response content. Screenshot: `tmp/utk-ai-prompt-history-smoke.png`.
- Browser validation sent `проверь логи ufr-kpnsb-product-request-workflow-service`; PostgreSQL confirmed two prompt-history rows linked to the same assistant message, sequences `1` and `2`, with purposes `initial-response` and `natural-language-tool-result-synthesis`. Screenshot: `tmp/utk-ai-prompt-history-agent-loop.png`.
- Browser validation reran `проверь логи ufr-kpnsb-limit-service`; confirmed highlighted intent `Проверю логи skp-email-service...`, downstream MCP search, and two prompt-history rows for one assistant message. Screenshot: `tmp/utk-ai-limit-service-downstream-intent.png`.
- Browser validation reran `проверь логи ufr-kpnsb-limit-service` after the strict JSON `intent` contract; confirmed primary summary stayed visible, raw `{"tool":"search_logs"}` JSON was not shown, the `intent` rendered as a highlighted callout, downstream MCP search executed, and prompt history stored `initial-response` plus `tool-result-synthesis`. Screenshot: `tmp/utk-ai-json-intent-tool-call.png`.
- Browser validation reran `проверь логи ufr-kpnsb-limit-service` after the render fix; confirmed exactly one `.chat-tool-intent` callout, normal error-message blockquotes with transparent background, no raw intent marker in text, and no console errors. Screenshot: `tmp/utk-ai-intent-only-highlight.png`.
- Browser validation reran `проверь логи ufr-kpnsb-limit-service` after moving error quote classification to the model; confirmed one blue `.chat-tool-intent`, two red `.chat-error-quote` blocks from model markers, no raw `UTK_ERROR_QUOTE` marker, and no console errors. Screenshot: `tmp/utk-ai-model-error-quote-red.png`.
- Browser validation sent `hi debug collapsible smoke` with the debug console expanded; confirmed three `.chat-debug-entry` items, zero open by default, and the first entry expanded after clicking its summary. Screenshot: `tmp/utk-ai-debug-collapsible-entries.png`.
- Browser validation sent `debug compact row smoke` with the debug console expanded; confirmed three collapsed entries, zero open by default, each row measured `20px` high, summaries measured `20px`, and log gap was `4px`. Screenshot: `tmp/utk-ai-debug-compact-collapsed.png`.
- Browser validation reran `проверь логи ufr-kpnsb-limit-service`; confirmed the primary-service analysis stayed visible before the highlighted `skp-email-service` JSON intent, downstream result followed, raw tool JSON was hidden, and no console errors appeared. Screenshot: `tmp/utk-ai-primary-preserved-intent-clean.png`.
- `dotnet ef migrations add AddChatDebugEntries --project src/Utk.Ai.Infrastructure/Utk.Ai.Infrastructure.csproj --startup-project src/Utk.Ai.Api/Utk.Ai.Api.csproj --output-dir Persistence/Migrations` created migration `20260508110125_AddChatDebugEntries`.
- Browser validation sent `debug persistence smoke`, reloaded the same conversation `a1941e8b-18f3-4515-b8d1-fc0b4d358ddf`, and confirmed the debug console restored three entries including the completion status. PostgreSQL confirmed three `chat_debug_entries` rows with sequence `1..3`. Screenshot: `tmp/utk-ai-debug-persistence-reload.png`.

## Current State

- Chat follow-ups asking for warnings now select the `log-search` skill and perform a real MCP search using the previously mentioned service context.
- The current log-search skill supports severity switching for common error, warning, debug, and information terms.
- Chat log search can query any explicitly named service through MCP/Kibana, even when that service is not listed in `Monitoring:Targets`.
- Chat log search no longer auto-runs downstream checks in the backend skill. It exposes downstream candidates to AlfaGen; the model must initiate downstream `search_logs` calls through the agent loop.
- Chat composer remembers the last 10 submitted prompts locally and supports Up/Down history browsing.
- Agent tool execution is centralized behind `IAgentToolExecutor`; `ChatHub` no longer owns `list_services` or `search_logs` execution logic.
- Chat log-search responses are internally sequenced: primary analysis is shown before downstream work, model intent is visible, downstream results refer back to the original error, and final output includes a compact error-flow visualization without exposing technical phase labels to the user.
- Chat highlights model downstream-check intent inline as a quote-style callout before the downstream MCP search executes.
- Chat prompt history now persists each AlfaGen request/response under the assistant message that caused it, so multi-step agent loops can be replayed or debugged by `chat_message_id` and `sequence`.
- Downstream intent detection now handles suggestive phrases such as `имеет смысл проверить`, preventing the model from stopping after a recommendation when it has enough context to run the downstream search.
- Downstream tool checks now rely on strict JSON with an `intent` field instead of prose intent detection; prose fallback was removed from the loop.
- Debug console messages are collapsed by default and their collapsed summaries are compact 20px log rows.
- Tool-call parsing preserves model prose around structured JSON; the app does not classify or remove arbitrary Russian/English prose outside the exact tool-call JSON block.
- Debug console output is persisted per conversation and replayed on history load, preserving append semantics for streamed chunks.
