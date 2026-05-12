---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-09
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-09 UTK AI Native Tool Calls

## Goal

- Switch `utk-ai` Chat from assistant-text JSON tool requests to native OpenAI chat tool calls.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`

## Actions

- Changed `ILlmChatClient` from plain streamed text to a `CompleteAsync` result that can carry `LlmToolCall` metadata.
- Updated `AlfaGenChatClient` to register available `AgentToolDescriptor` entries as OpenAI `ChatTool` definitions and accumulate streamed `ToolCallUpdates`.
- Replayed tool-call history with SDK-native assistant `tool_calls` messages and `tool` result messages keyed by `tool_call_id`.
- Reworked `ChatToolLoopRunner` to execute native tool calls, append native tool result messages, synthesize after tool execution, and force a final answer once the max tool iteration limit is reached.
- Removed `ChatToolRequestReader`, malformed assistant-text JSON retry handling, and executable assistant-text tool parsing.
- Reduced and renamed the old parser into `ChatToolCallDebugFormatter`, which now only formats debug output.
- Removed the one-method `ChatToolLoopPromptBuilder`; the max-iteration prompt now lives in `ChatToolLoopRunner`.
- Updated chat prompts and `docs/refactoring-plan.md` to describe native tool calls rather than strict JSON tool requests.
- Updated Application tests around native tool-call flow, prompt history requests, max tool iterations, and tool-call debug text.
- Added `search_aggregated_logs` as an available native agent tool for bucketed Kibana patterns, backed by infrastructure `SearchAggregatedLogsAgentTool` and the existing MCP aggregation client.
- Shared service-target resolution between raw and aggregated log-search tools through `AgentToolTargetResolver`.
- Updated tool schema generation so OpenAI tool definitions can advertise integer parameters and the `groupBy` string-array parameter.
- Updated Application and Contracts tool catalogs plus log-search prompts so the model can choose `search_logs` for exact samples or `search_aggregated_logs` for repeated-error summaries.
- Carried `kibanaUrl` from `utk-mcp` into `KibanaLogEntry` and included it in backend-selected log context plus raw/aggregated downstream tool result text.
- Updated log-skill evidence rules and the system prompt so concrete log answers include relevant Kibana evidence links when MCP returns them.
- Added formatter coverage to assert backend log context includes `documentId` and `kibanaUrl`.
- Simplified monitoring link generation by carrying `KibanaLogEntry.KibanaUrl` into `AnalyzedErrorEvidence` and setting `ErrorFinding.RepresentativeKibanaUrl` from MCP-provided evidence links.
- Removed local monitoring Discover URL construction helpers and hardcoded Kibana data-view/base URL constants from `MonitoringRunProcessor`.

## Validation

- `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers` passed.
- `dotnet test Utk.Ai.slnx --no-build --disable-build-servers` passed.
- `git diff --check` passed.
- Confirmed stale symbols such as `ChatToolRequestReader`, `StreamResponseAsync`, `malformed-tool`, and text-protocol references no longer appear in `src`, `tests`, or docs.
- Re-ran `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers`, `dotnet test Utk.Ai.slnx --no-build --disable-build-servers`, and `git diff --check` after adding `search_aggregated_logs`; all passed.
- Re-ran `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers`, `dotnet test Utk.Ai.slnx --no-build --disable-build-servers`, and `git diff --check` after adding `kibanaUrl` evidence propagation; all passed.
- Re-ran `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers`, `dotnet test Utk.Ai.slnx --no-build --disable-build-servers`, and `git diff --check` after simplifying monitoring link generation; all passed.
- Restarted the local `utk-ai` API tmux pane; `GET http://127.0.0.1:9596/health` returned `200`.
- Review pass re-ran `dotnet build Utk.Ai.slnx --no-restore --disable-build-servers`, `dotnet test Utk.Ai.slnx --no-build --disable-build-servers`, and `git diff --check`; all passed.

## Current State

- The chat agent loop now depends on native OpenAI tool-call metadata instead of parsing assistant-visible text into executable tool requests.
- Chat can now call `search_aggregated_logs` for top repeated errors or broad summaries using `bucketLimit`, `examplesPerBucket`, and optional `groupBy`.
- Chat log context and tool results now expose MCP `kibanaUrl` values to the model; final answer behavior is prompt-enforced and still needs a live chat smoke test.
- Monitoring finding representative links now depend on MCP `kibanaUrl`; runs against an older MCP without `kibanaUrl` will persist findings without representative links rather than synthesizing local links.
- `ChatToolLoopRunner` still coordinates visible preface output, visible tool intent, debug events, tool execution, tool-result history, synthesis, and max-iteration fallback.
- `ChatToolCallDebugFormatter` remains as a small debug-formatting helper; it no longer parses or detects JSON tool-call text.
- Live AlfaGen endpoint behavior with native tools has not been browser-smoke-tested in this session; verification was build/test/diff hygiene only.

## Decisions

- Keep tool execution centralized behind `IAgentToolExecutor`.
- Keep SignalR/ChatHub transport-only; native tool-call orchestration remains in Application.
- Prefer native OpenAI tool calls over model-printed JSON because the protocol becomes structured at the SDK/API boundary and eliminates malformed JSON recovery branches.
- Keep raw and aggregated Kibana searches as separate tool IDs so the model can choose between exact samples and grouped patterns explicitly.
- Require log answers to carry Kibana evidence links from MCP-provided `kibanaUrl` values when citing concrete errors.
- Keep outbound Kibana Discover URL generation centralized in `utk-mcp`; `utk-ai` monitoring consumes links from evidence instead of encoding Kibana URL/Rison details.

## Follow-Up

- Consider adding a narrow infrastructure test around `AlfaGenChatClient` tool schema/message conversion if the SDK types can be exercised without live network calls.
- Browser-smoke downstream `search_logs` and aggregated `search_aggregated_logs` chat flows against the real local stack, including whether final answers preserve `kibanaUrl` evidence links.
- Harden the max-tool-iteration final request so tools are disabled or ignored if AlfaGen requests another tool after the limit.
- Handle invalid native tool-call argument JSON without failing the whole chat turn.
