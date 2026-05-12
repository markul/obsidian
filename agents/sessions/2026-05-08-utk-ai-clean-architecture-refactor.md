---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-05-08
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
project: "[[agents/projects/utk-ai|UTK AI]]"
ticket:
---

# 2026-05-08 UTK AI Clean Architecture Refactor

## Goal

- Improve `utk-ai` Clean Architecture boundaries while preserving current chat, monitoring, SignalR, and Blazor behavior.

## Scope

- Agent project: [[agents/projects/utk-ai|UTK AI]]
- Repo: `/home/marat/dev/git/alfa/utk-ai`

## Actions

- Added `Utk.Ai.Contracts` as a shared DTO/catalog assembly for HTTP API, SignalR, and Web client wire types.
- Removed the Blazor WebAssembly dependency on `Utk.Ai.Application`; Web now references only `Utk.Ai.Contracts`.
- Moved monitoring read/query logic out of `MonitoringApi` into the Application `IMonitoringQueryService` port with an Infrastructure `MonitoringQueryService` EF implementation.
- Kept API endpoints as transport adapters for monitoring reads, trigger requests, investigation queue/status, and plain Kibana investigations.
- Mapped SignalR monitoring completion/progress payloads to Contracts DTOs before broadcasting.
- Moved the agent tool catalog used by the Web/sidebar and Chat prompt into Contracts; unavailable tools are now marked `Not wired` and excluded from prompt summaries/tool parsing.
- Fixed final investigation progress ordering so `Done`/`Error` is saved to PostgreSQL before SignalR progress is published.
- Made MCP VPN preflight disabled in base config and enabled only in Development config.
- Updated API and Web Dockerfiles to restore/build the new Contracts project.
- Extracted chat-agent orchestration from `ChatHub` into Application `ChatAgentService` behind `IChatAgentService` and `IChatAgentClient`.
- Reduced `ChatHub` to a SignalR adapter that forwards `SendMessage`/`GetHistory` to Application and maps output-port events to SignalR messages.
- Decomposed the oversized `ChatAgentService` by extracting `ChatLogContextBuilder`, `ChatToolCallParser`, `ChatLogQueryHeuristics`, and `ChatResponseFormatting`.
- Registered `IChatLogContextBuilder` in Infrastructure DI; chat log context assembly now persists debug entries through `IChatHistoryService` while keeping SignalR-specific types out of Application.
- Further reduced `ChatAgentService` by extracting prompt/history assembly into `ChatMessageBuilder` and the AlfaGen/tool-call execution loop into `ChatAgentTurnRunner`.
- Removed unused chat helper leftovers `ChatLogFormatting` and `ChatText`.
- Fixed the architecture review concerns: Application no longer references `Utk.Ai.Contracts`; API/SignalR map wire DTOs at the transport edge; chat progress/debug/chunk output is centralized through `ChatTurnEventSink`; enum conversions now use explicit switch mappings.
- Made persisted chat completion final by swallowing/logging completion-notification failures after the assistant message has already been saved.

## Validation

- `dotnet build Utk.Ai.slnx` passed.
- `dotnet test Utk.Ai.slnx --no-build` passed.
- `git diff --check` passed.
- Confirmed `src/Utk.Ai.Web` no longer references `Utk.Ai.Application` or the deleted `Utk.Ai.Web.Models` namespace.
- Confirmed SignalR types and `Hub` dependencies are confined to `src/Utk.Ai.Api/Realtime/ChatHub.cs`, not Application chat orchestration.
- Restarted the local API pane on `http://localhost:9596`; `/health` returned `Healthy`.
- After decomposition, `dotnet build Utk.Ai.slnx`, `dotnet test Utk.Ai.slnx --no-build`, and `git diff --check` passed again.
- Restarted the local API pane on `http://localhost:9596` after the decomposition; `/health` returned `Healthy`.
- After the second decomposition pass, `dotnet build Utk.Ai.slnx`, `dotnet test Utk.Ai.slnx --no-build`, and `git diff --check` passed again.
- Restarted the local API pane on `http://localhost:9596`; `/health` returned `Healthy`.
- After fixing review concerns, `dotnet build Utk.Ai.slnx`, `dotnet test Utk.Ai.slnx --no-build`, and `git diff --check` passed.
- Restarted the local API pane on `http://localhost:9596`; `/health` returned `Healthy`.

## Current State

- Clean Architecture dependency direction is stricter: Web uses Contracts, API uses Application/Infrastructure/Contracts, Application defines query and chat-agent ports, and Infrastructure owns EF/external adapter implementations.
- Chat-agent orchestration now lives in Application. `ChatHub` is transport-only SignalR glue.
- `ChatAgentService` is now a lifecycle coordinator for send/history/completion/failure handling; prompt construction, log-context construction, model/tool-loop execution, tool-call parsing, log-query heuristics, and visible response formatting are separated into focused Application services/helpers.
- Application owns its use-case models for chat and monitoring queries; `Utk.Ai.Contracts` is now used by Web/API transport surfaces rather than Application/Infrastructure.
- Chat event emission and debug persistence are centralized through `ChatTurnEventSink`.
