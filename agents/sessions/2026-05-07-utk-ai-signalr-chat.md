---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-07
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-07 UTK AI SignalR Chat

## Goal

- Add a Web chat page that talks to the configured AlfaGen/OpenAI-compatible LLM over SignalR transport.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`
- Related notes:
  - [[daily/2026-05-07]]
  - [[agents/sessions/2026-05-06-utk-ai-investigation-agent-and-plain-search|2026-05-06 UTK AI Investigation Agent And Plain Search]]

## Actions

- Added `Utk.Ai.Application.Chat.ILlmChatClient` and a small chat message contract.
- Added `AlfaGenChatClient`, reusing the existing `AlfaGenOptions` and official OpenAI .NET SDK streaming chat API.
- Registered the chat client in Infrastructure dependency injection.
- Added API SignalR hub `/hubs/chat` with `SendMessage`, `ChatStarted`, `ChatChunk`, `ChatCompleted`, and `ChatFailed` messages.
- Added Blazor WebAssembly `/chat` page that connects to `/hubs/chat`, sends user prompts through SignalR, and renders streamed assistant chunks.
- Added a sidebar Chat icon link matching the existing thin/collapsed sidebar style.
- Reduced API console noise by raising Microsoft/EF Core log categories to warnings and suppressing EF SQL command logging.
- Added concise AlfaGen lifecycle logs for finding analysis, issue-investigation planning, issue-investigation synthesis, and chat streaming. Logs include model, IDs/counts, duration, response size, and failure status without prompt text or secrets.
- Fixed misleading chat "stuck streaming" behavior by logging the SignalR hub boundary, using `Context.ConnectionAborted` instead of a hub method `CancellationToken`, adding a configurable `AlfaGen:ChatTimeoutSeconds`, and marking the assistant message as streaming only after the server sends `ChatStarted`.
- Added repo-root Playwright setup for Web E2E tests: `package.json`, `package-lock.json`, `playwright.config.ts`, ignored Playwright output folders, and `e2e/chat.spec.ts`.
- Added stable `data-testid` hooks to the chat UI for Playwright.
- Redesigned the chat page toward a simplified ChatGPT-like UI: centered conversation column, assistant/user avatar rows, quieter header, empty-state prompt, rounded bottom composer, and icon-only send button.
- Added Enter-to-send in the chat composer while preserving Shift+Enter for newlines.
- Constrained the chat page to the viewport and moved long-response overflow into the transcript area so the composer and sidebar stay visible.
- Restored the composer to the earlier reference style from `tmp/utk-ai-chat-redesign-message.png`: shadowed rounded container, borderless input, and circular arrow send button.
- Fixed the remaining reference mismatch by removing shadow clipping from the chat surface; the composer shadow now fades like the saved reference while the transcript still owns long-response scrolling.
- Extended the reserved space below the composer so the bottom shadow is fully visible instead of clipped by the viewport edge.
- Matched the composer/shadow to the saved reference by removing `.chat-page` overflow clipping and restoring the original `0 8px 24px` shadow after verification.
- Moved chat scrolling from the inner transcript area to the main content container at the window edge, and added JS-assisted auto-scroll after user messages, streaming chunks, completion, and failure.
- Restarted the local API and Web panes in tmux using stable `dotnet run`.
- Added persisted chat history: `ChatConversation` and `ChatMessage` domain entities, `IChatHistoryService`, EF mappings, and migration `20260507095615_AddChatHistory`.
- Updated `ChatHub` to load persisted conversation messages for LLM context, store user messages immediately, store assistant placeholders at stream start, and persist completed or failed assistant responses.
- Added `GetHistory` to the chat SignalR hub and updated the Web `ChatSessionState` to keep a stable conversation ID in browser `localStorage` and reload stored messages on startup.
- Extended the Playwright chat smoke test so it sends one real message, waits for the streamed response, reloads the page, and verifies the same persisted history is rendered.

## Decisions

- Use SignalR as the chat transport for both prompt submission and streamed LLM responses; no REST endpoint was added for chat.
- Persist the current browser conversation server-side while keeping chat transport SignalR-only; the browser owns the active conversation ID through `localStorage`.
- Reuse the existing AlfaGen model configuration instead of introducing separate chat-specific endpoint/model settings.
- Keep console logging focused on application and AlfaGen progress; do not print EF SQL commands during worker polling.

## Validation

- `dotnet build Utk.Ai.slnx` passed.
- `curl -sS -o /tmp/utk-ai-health.out -w '%{http_code}' http://127.0.0.1:9596/health` returned `200`.
- `curl -sS -o /tmp/utk-ai-chat-page.out -w '%{http_code}' http://127.0.0.1:9595/chat` returned `200`.
- `curl -sS -X POST -o /tmp/utk-ai-chat-negotiate.out -w '%{http_code}' 'http://127.0.0.1:9596/hubs/chat/negotiate?negotiateVersion=1'` returned `200`.
- `dotnet build Utk.Ai.slnx` passed after logging changes.
- API was restarted, `/health` returned `200`, and idle worker polling no longer emitted new EF SQL command logs in the tmux pane.
- Temporary .NET SignalR client test against `/hubs/chat` sent `hi` and received `ChatStarted`, streamed `ChatChunk` events, and `ChatCompleted`; a short 15-second client test disconnected before AlfaGen returned, confirming the apparent UI stall was slow first-token/response latency rather than a dead API.
- `npm install` completed and `npm run test:e2e -- --list` discovered the chat test.
- `npm run test:e2e` originally reproduced the UI bug: API logs showed chat chunks/completion or timeout, but the Playwright assertion saw an empty assistant message body until failure. Artifacts were written under ignored `test-results/`.
- `dotnet build Utk.Ai.slnx` passed after the chat redesign.
- Manual Playwright browser script confirmed `/chat` loads with status `Connected`, the empty state renders, the composer is centered at 860px width on desktop, and a submitted `hi` message renders user and assistant rows.
- Manual Playwright browser scripts confirmed plain Enter sends a message and clears the input, while Shift+Enter inserts a newline.
- Manual Playwright layout check injected a 180-line assistant response and confirmed document/sidebar height stayed at the viewport, the composer remained visible, and the transcript became scrollable.
- Manual browser layout check confirmed the restored composer has an 8px radius, shadow, borderless input, and 40px circular send button. Screenshot: `tmp/utk-ai-chat-composer-reference-style.png`.
- Follow-up browser check confirmed the un-clipped reference-style composer and retained long-response behavior: document/sidebar stay viewport-height, composer visible, transcript scrollable. Screenshot: `tmp/utk-ai-chat-composer-reference-style-v3.png`.
- Browser check confirmed 74px of visible space below the composer while long-response transcript scrolling still works. Screenshot: `tmp/utk-ai-chat-composer-shadow-extended.png`.
- Pixel/metric comparison against `tmp/utk-ai-chat-redesign-message.png` now reports the same composer/shadow bounding box `(308, 767, 1190, 853)`. Screenshot: `tmp/utk-ai-chat-composer-shadow-final.png`.
- Browser layout check with a 180-line assistant response confirmed `.app-content` is the scroll container at the right edge, `.chat-transcript` has no own scrollbar, the composer remains visible, and the sidebar stays viewport-height. Screenshot: `tmp/utk-ai-chat-main-edge-scroll.png`.
- Restyled the chat page closer to the saved ChatGPT light/dark references: theme-aware user bubble, plain assistant text, transparent composer tool icons, `Instant` mode label, microphone icon, and theme-aware circular send control.
- Manual Playwright screenshots confirmed the light and dark visual states. Screenshots: `tmp/utk-ai-chat-chatgpt-style-light.png` and `tmp/utk-ai-chat-chatgpt-style-dark.png`.
- Simplified the chat page after review by removing the visible top toolbar, plus control, `Instant` label, microphone control, and surrounding top/bottom padding bands; the composer now uses the `Message UTK AI` placeholder with only the send button.
- Manual Playwright check confirmed the simplified page has no visible toolbar or extra composer controls and fills the viewport from top to bottom. Screenshot: `tmp/utk-ai-chat-clean-simple.png`.
- Added Markdown rendering for assistant chat responses with Markdig, disabling raw HTML and styling headings, paragraphs, lists, tables, inline code, and fenced code blocks to match the ChatGPT-style response layout.
- Manual Playwright visual check confirmed Markdown renders as structured HTML instead of literal `#`, `**`, table pipes, or fenced-code markers. Screenshot: `tmp/utk-ai-chat-markdown-rendering.png`.
- Reduced the chat composer height and pinned it 16px above the viewport bottom during main-window scrolling. Screenshot: `tmp/utk-ai-chat-pinned-small-composer.png`.
- Added a three-dot waiting indicator for empty assistant responses while the app waits for the first streamed chunk, and increased the composer bottom gap to 32px. Screenshot: `tmp/utk-ai-chat-waiting-indicator.png`.
- Moved chat messages, draft input, connection state, and SignalR hub ownership into a scoped Web `ChatSessionState` service so navigating between pages preserves the current chat state and does not dispose an in-flight response.
- Manual Playwright navigation check confirmed a typed chat draft survives switching from Chat to Monitoring and back. Screenshot: `tmp/utk-ai-chat-state-preserved.png`.
- Made Chat the default Web page at `/` while preserving `/chat` as an alias, moved Monitoring to `/monitoring`, and updated sidebar navigation targets.
- Manual Playwright route check confirmed `/` renders Chat, `/monitoring` renders Monitoring, and sidebar buttons navigate to the new routes.
- Added scoped `MonitoringPageState` for `/monitoring` so filters, loaded findings, selected finding, investigation state, and monitoring toasts survive same-tab navigation away and back.
- Manual Playwright navigation check confirmed the Monitoring service filter stays selected after `Monitoring -> Chat -> Monitoring`. Screenshot: `tmp/utk-ai-monitoring-state-preserved.png`.
- `dotnet build Utk.Ai.slnx` passed after the chat log-query fixes and `ChatProgress` event wiring.
- Runtime logs confirmed a Russian `за вчера` query for `skp-product-change-workflow-service` searched `2026-05-06 00:00:00Z` to `2026-05-07 00:00:00Z` and returned `TotalHits=10000`, `ReturnedHits=20`.
- API and Web were restarted; `curl` checks returned `api:200` for `http://localhost:9596/health` and `web:200` for `http://localhost:9595/`.
- `npx playwright test e2e/chat.spec.ts` passed after the final chat SignalR/UI updates.
- Added backend-assisted log context for Chat: when the prompt looks log-related and contains exact configured service names from API `Monitoring:Targets`, `ChatHub` queries Kibana error logs through `IKibanaLogClient` for the recent UTC window and injects sampled log context before calling AlfaGen.
- Chat now includes the configured service list in its system context and asks for a configured service when a log query does not name one.
- Fixed the chat log-query path after AlfaGen answered that it had no Kibana access despite injected log context: `за вчера`/`yesterday` now uses previous UTC-day boundaries, the system prompt explicitly treats retrieved Kibana context as available evidence, zero-hit/search-failure behavior is explicit, and API logs total/returned hits for chat log searches.
- Added a lightweight `ChatProgress` SignalR event so the chat UI can show backend progress such as `Searching Kibana logs...` before the first model chunk arrives.
- `dotnet build Utk.Ai.slnx` passed after adding chat history persistence and migration.
- API restart applied the chat history migration; runtime logs showed `Chat history loaded` with `Messages=2` after the Playwright reload.
- `npx playwright test e2e/chat.spec.ts` passed with the new reload-persistence assertion.
- Investigated conversation `9570d824-20a8-4675-9c90-8bc08471b7a7` after AlfaGen answered "no errors" despite API logs showing `TotalHits=10000`, `ReturnedHits=20` for `skp-product-change-workflow-service` on `2026-05-06`.
- Fixed chat log-query prompt construction so live Kibana context includes an explicit `Search result verdict: ERRORS_FOUND` when hits are returned and is embedded directly in the current user turn; for log queries, old assistant history is no longer sent to the model.
- Validation in the same persisted conversation now reports `Total hits: ≥10,000 errors` and identifies Kafka topic authorization/SASL authentication failures instead of saying no errors or no Kibana access.
- Added a reusable `AgentToolCatalog` in `Utk.Ai.Application.AgentTools` as the shared source for agentic tool metadata.
- Added a sidebar `Available tools` info button above the theme button; it opens a modal listing tool IDs, categories, availability, descriptions, and inputs from the shared catalog.
- Added the Application project reference to the Web client so the UI reads the same catalog as API/agent code.
- `dotnet build Utk.Ai.slnx` passed after adding `AgentToolCatalog` and the Web sidebar modal.
- Browser check confirmed the sidebar tools modal opens and includes `search_logs`, `plain_investigate`, and `load_monitoring_findings`. Screenshot: `tmp/utk-ai-tools-modal.png`.
- Suppressed expected `OperationCanceledException` logs when Plain search or Monitoring SignalR progress connections are canceled during navigation/disposal.
- Browser check confirmed navigating from `/plain-search` to Chat no longer emits `Plain search progress unavailable: OperationCanceled` in the console.
- Split Monitoring page loading state from trigger-monitoring state so opening `/monitoring` does not disable the `Trigger monitoring` button.
- Changed disabled action-button cursor styling from `wait` to `default` so temporarily unavailable buttons do not look broken.
- Reworked the chat composer so the send button calls the Blazor send handler directly instead of relying on form submit, restored the Enter-to-send binding for newline prevention, and made the disabled send button visibly muted.
- Added history-aware service resolution for chat log queries: if the current log prompt does not name an exact configured service, `ChatHub` scans recent persisted/client chat context for focused service mentions and ignores broad service catalog/list responses.
- Added environment hint handling for referential log prompts such as `dev`/`дев`, `test`/`тест`, and `prod`/`прод` when resolving the remembered service target.
- `dotnet build Utk.Ai.slnx` passed after the history-aware service resolution change.
- Runtime validation against conversation `f6bea9fc-aee7-4f9e-b228-b2c917762b16` confirmed the prompt `проверь логи за вчера по сервису который я искал в дев` inferred `ufr-kpnsb-limit-service` from prior chat context, searched dev Kibana for `2026-05-06 00:00:00Z` to `2026-05-07 00:00:00Z`, and returned `TotalHits=2`.
- Added a `ChatDebug` SignalR event around the AlfaGen chat call so the Web client can observe the full message list sent to AlfaGen, streamed response chunks, and completion/failure status in real time.
- Added a collapsed-by-default bottom debug console on the Chat page; when expanded, it uses a black background with white monospace text and occupies 30% of the viewport height.
- `dotnet build Utk.Ai.slnx` passed after adding the Chat debug console.
- Manual Playwright browser check confirmed the debug console starts collapsed, expands to 270px at a 900px viewport, receives the sent AlfaGen request messages plus streamed response/status, and has no browser console errors. Screenshot: `tmp/utk-ai-chat-debug-console.png`.
- Hardened chat composer input state by replacing direct `@bind` to scoped `ChatSessionState.DraftMessage` with an explicit `SetDraftMessage` path that notifies state changes on every input event.
- Browser validation confirmed the send button starts disabled, becomes enabled after typing in Chrome/Chromium, and can be clicked without console errors.
- Fixed expanded debug-console layout so the chat page reserves the lower 30% for the console; the transcript scrolls in the upper area and the composer remains visible above the console.
- Synthetic long-conversation browser check confirmed the composer stays 32px above the debug console, transcript overflow is scrollable, and page-level scrolling does not push the composer behind the console. Screenshot: `tmp/utk-ai-chat-debug-console-long-conversation.png`.
- Reworked the chat debug console from a bottom dock into a right-side column. Collapsed mode is a 40px side rail; expanded mode splits the page into chat and console columns with independent transcript/debug scrolling.
- Browser layout check confirmed the expanded right console does not overlap the composer, the collapsed rail stays 40px wide, and both columns scroll independently. Screenshot: `tmp/utk-ai-chat-debug-side-column.png`.
- Moved the chat two-column split to fill `app-content` instead of the centered `page-shell` by removing chat-route shell max-width and padding.
- Browser geometry check confirmed `page-shell` and `chat-main` match `app-content`, and the debug column aligns to the app-content right edge. Screenshot: `tmp/utk-ai-chat-debug-side-column-app-content.png`.
- Made the collapsed debug-console rail flush with the right window edge by removing its right border and right-side radius.
- Browser check confirmed collapsed rail `rightGap=0`, `borderRightWidth=0px`, and right radii are `0px`. Screenshot: `tmp/utk-ai-chat-debug-collapsed-flush-right.png`.
- Added a `New chat` button in the chat toolbar. It creates a fresh browser conversation ID, stores it in `localStorage`, clears current chat messages, clears AlfaGen debug entries, resets the draft, and keeps the existing SignalR connection.
- Browser validation confirmed `New chat` changes the stored conversation ID, clears two existing messages and debug output, shows the empty chat state, and leaves send disabled until new text is entered. Screenshot: `tmp/utk-ai-chat-new-session.png`.
- Added green `mcp request` and `mcp response` debug entries around chat log searches through `utk-mcp`; entries include environment, services, UTC window, severity, limit, total hits, returned hits, and up to five sampled hit summaries.
- Added yellow `tool request` debug highlighting for model responses that appear to request execution of a known `AgentToolCatalog` tool.
- Browser validation with `ошибки в логах ufr-kpnsb-limit-service за вчера` confirmed the debug console showed green MCP request/response entries for `search_logs`, including `TotalHits=2`; a separate DOM style check confirmed yellow tool-request coloring. Screenshot: `tmp/utk-ai-chat-debug-mcp-green.png`.
- Implemented a first-pass backend agent loop for Chat: AlfaGen may return a structured tool-call JSON object, the hub intercepts it without sending it to the chat transcript, executes the tool, feeds the tool result back to AlfaGen, then streams only the final answer to the user.
- Implemented executable chat tools for `list_services` and `search_logs`; `search_logs` uses configured monitoring targets and `IKibanaLogClient`, emitting green MCP debug entries around the call.
- Browser validation with `Use the list_services tool before answering. What services can you inspect?` confirmed the tool-call JSON did not appear in chat, a yellow `tool request` debug entry was shown, and the final chat answer listed available services. Screenshot: `tmp/utk-ai-chat-agent-tool-loop.png`.
- Fixed chat auto-scroll after the right-side debug-console split by targeting the chat transcript scroll container instead of the non-scrolling `.app-content` shell.
- Browser validation confirmed `utkChatComposer.scrollChatToBottom()` scrolls `[data-testid="chat-transcript"]` to the bottom while `.app-content` remains fixed. Screenshot: `tmp/utk-ai-chat-autoscroll-transcript.png`.
- Analyzed latest conversation `af901de7-273d-4a98-b03c-b3014d47b8d2`; AlfaGen emitted `[TOOL_CALL] {tool => "search_logs", ...} [/TOOL_CALL]`, which was not strict JSON, so the hub streamed and persisted it as assistant text instead of executing `search_logs`.
- Hardened chat tool-call handling so the hub normalizes AlfaGen pseudo tool-call syntax with `=>`, accepts `[TOOL_CALL]` wrappers, keeps tool-looking text out of the chat transcript, retries once with a strict-JSON instruction when parsing fails, and reports malformed tool calls only in the debug console.
- Improved log follow-up handling so prompts such as `do it` or `the environment is dev` can trigger Kibana log context using recent chat history, and tightened environment detection so `kibana-test` inside a URL does not by itself force the `test` environment.
- Reflection validation confirmed the exact malformed tool-call shape from the failed conversation now parses as `search_logs`, and `ResolveRequestedEnvironment("https://kibana-test/...")` returns no environment while `the environment is dev` returns `dev`.
- `dotnet build Utk.Ai.slnx` passed after hardening malformed tool-call parsing and log follow-up detection.
- Temporary parser check under repo `tmp/` confirmed the failed `[TOOL_CALL] {tool => ...}` format parses to executable `search_logs`; the temp project was removed after validation.
- API was restarted on `http://localhost:9596`; `/health` returned `200`.
- `npx playwright test e2e/chat.spec.ts` passed after the tool-call parsing changes.
- Added the active chat conversation ID to the right-side debug-console title; expanded mode shows a short ID with the full UUID in the tooltip.
- Browser validation confirmed the debug-console header rendered `AlfaGen Debug {short-id}` and exposed the full `Conversation {uuid}` tooltip. Screenshot: `tmp/utk-ai-chat-debug-conversation-id.png`.
- Added `@` service mentions to the chat composer. Typing `@` opens a filtered service picker backed by the shared Monitoring service list, and selecting a service inserts the service name into the prompt.
- Browser validation confirmed `@limit` ranks `ufr-kpnsb-limit-service` first, inserts it into the composer, closes the picker, and produces no console errors. Screenshot: `tmp/utk-ai-chat-service-mention-picker.png`.
- Extended `@` service mentions to support hyphenated shorthand such as `@pr-wo-ser`; query segments match service-name segments by prefix, and acronym-like groups such as `pr` rank `product-request` above looser `product` matches.
- Browser validation confirmed `@pr-wo-ser` ranks and inserts `ufr-kpnsb-product-request-workflow-service` first. Screenshot: `tmp/utk-ai-chat-service-mention-shorthand.png`.
- Added keyboard navigation for chat service mentions: Up/Down moves the active service, Enter inserts it, and Escape closes the picker. The global Enter-to-send helper now prevents textarea defaults while the picker is open without sending the chat message.
- Browser validation confirmed Down/Enter selects the second `@pr-wo-ser` option without sending the message or adding a newline, and Up wraps from the first option to the last. Screenshot: `tmp/utk-ai-chat-service-mention-keyboard.png`.
- Added a first declarative agent skill catalog with backend-owned `log-search` skill metadata: trigger rules, required inputs, allowed tools, and evidence rules.
- Integrated `log-search` into ChatHub's direct log-query path. When selected, the hub emits a `skill` debug entry, includes selected-skill metadata in the LLM prompt, executes the existing Kibana/MCP search path, and still lets AlfaGen only summarize retrieved evidence.
- Styled `skill` debug entries separately from MCP/tool entries so selected backend skills are visually distinct in the debug console.
- Browser validation with `find errors in skp-product-change-workflow-service last 24 hours` confirmed the debug console shows `skill: log-search`, followed by `utk-mcp tool call: search_logs` and `utk-mcp tool result: search_logs`, with a distinct skill debug entry. Screenshot: `tmp/utk-ai-chat-log-search-skill.png`.

## Current State

- API is running from `src/Utk.Ai.Api` on `http://localhost:9596`.
- Web is running from `src/Utk.Ai.Web` on `http://localhost:9595`.
- Chat page is available at `http://localhost:9595/chat`.
- API console logging is quieter by default and should surface AlfaGen start/completion/failure messages for monitoring analysis, investigations, and chat.
- Chat requests now emit hub receive/start/complete/timeout/error logs and should fail visibly after the configured timeout instead of remaining indefinitely in streaming state.
- Playwright is available for browser checks; the chat smoke test now passes against the running local API/Web pair.
- Chat page now visually matches a simplified assistant chat rather than a framed operations panel.
- Chat page now uses ChatGPT-like light/dark styling for message bubbles and a simplified bottom composer; after the right-side debug-console split, auto-scroll targets the transcript container while `.app-content` stays fixed.
- Assistant responses now render Markdown using Markdig in the Web client.
- Chat page state now survives same-tab navigation because the conversation is owned by a scoped Web service rather than the page component.
- Chat history now persists across page reloads for the active browser conversation using PostgreSQL-backed `chat_conversations` and `chat_messages`.
- Chat is the default route; Monitoring lives at `/monitoring`.
- Monitoring page state now survives same-tab navigation via scoped Web state.
- Chat can query Kibana error logs for configured services named in the prompt; the default window is the last 24 hours, simple hour parsing such as `24h` or `last hour` is supported, and `за вчера`/`yesterday` maps to the previous UTC day.
- Chat can infer the service for follow-up log questions from recent focused conversation context, so a user can first identify a service and then ask to check logs without repeating the exact service name.
- Chat can now treat short follow-ups such as `do it` or environment corrections as continuations of a recent log request when recent chat history contains log/tool context.
- For log-query prompts, current Kibana context is embedded into the active user turn and takes precedence over persisted assistant history to avoid repeating stale or incorrect prior answers.
- Chat tool calls are executed only at the hub boundary; malformed tool-looking model output is withheld from the transcript, logged in the debug console, and retried once as strict JSON.
- Chat shows backend progress while waiting for the first assistant chunk, including Kibana search status for log queries.
- Chat now has an optional right-side debug console for live AlfaGen interaction inspection; it is collapsed by default and streams request/response debug events over SignalR.
- The debug-console title shows the active conversation ID for easier DB/debug correlation.
- The chat composer supports `@` service selection using the same service list as Monitoring.
- Chat now has an explicit backend-owned `log-search` skill for log/error/Kibana prompts; the skill makes the selected behavior visible in debug output before MCP calls.
- Agentic tool metadata now has a shared catalog covering service discovery, log search, plain investigation, loading persisted plain investigations, loading monitoring findings, and monitoring finding investigations.
- Live LLM message completion still depends on the running API having a valid AlfaGen API key in local ignored configuration.
