---
tags:
  - agents/project
note-type: agent-project
service: "[[work/alfa-bank/services/utk-ai|utk-ai]]"
status: active
created: 2026-04-28T18:46
completed:
ticket:
---

# UTK AI

## Status

- `active`

## Timestamps

- Created: `2026-04-28 18:46`
- Completed: `not completed`

## Goal

- Plan and then implement `utk-ai` as a `.NET 10` web application for AI-assisted operational workflows, starting with Kibana error monitoring backed by `utk-mcp`, AlfaGen OpenAI-compatible models, and PostgreSQL

## Plan

1. Add `utk-mcp` Kibana log retrieval for configured service/environment targets.
2. Add AlfaGen OpenAI-compatible analysis flow and persist clustered findings/evidence.
3. Add scheduling and harden the development API/UI around real monitor output.

## Scope

- Repository: `utk-ai`
- Local path: `/home/marat/dev/git/alfa/utk-ai`
- Planned runtime: `.NET 10` ASP.NET Core API plus standalone Blazor WebAssembly UI
- Planned first feature: Kibana error monitoring with LLM aggregation, PostgreSQL persistence, and a standalone Blazor WebAssembly UI
- Planned local dev shape: Skaffold dependencies profile for PostgreSQL, with API and Web run locally
- Related notes:
  - [[agents/sessions/2026-04-28-utk-ai-bootstrap|2026-04-28 UTK AI Bootstrap]]
  - [[agents/sessions/2026-04-28-utk-ai-initial-plan|2026-04-28 UTK AI Initial Plan]]
  - [[agents/sessions/2026-04-29-utk-ai-monitoring-prototype|2026-04-29 UTK AI Monitoring Prototype]]
  - [[agents/sessions/2026-05-04-utk-ai-local-api-startup|2026-05-04 UTK AI Local API Startup]]
  - [[agents/sessions/2026-05-04-utk-ai-web-theme|2026-05-04 UTK AI Web Theme]]
  - [[agents/sessions/2026-05-04-utk-ai-async-monitoring|2026-05-04 UTK AI Async Monitoring]]
  - [[agents/sessions/2026-05-05-utk-ai-aggregation-persistence|2026-05-05 UTK AI Aggregation Persistence]]
  - [[agents/sessions/2026-05-05-utk-mcp-kibana-aggregation|2026-05-05 UTK MCP Kibana Aggregation]]
  - [[agents/sessions/2026-05-06-utk-ai-monitoring-config-and-classification|2026-05-06 UTK AI Monitoring Config And Classification]]
  - [[agents/sessions/2026-05-06-utk-ai-investigation-agent-and-plain-search|2026-05-06 UTK AI Investigation Agent And Plain Search]]
  - [[agents/sessions/2026-05-07-utk-ai-signalr-chat|2026-05-07 UTK AI SignalR Chat]]
  - [[agents/sessions/2026-05-08-utk-ai-chat-warning-followup|2026-05-08 UTK AI Chat Warning Followup]]
  - [[agents/sessions/2026-05-08-utk-ai-clean-architecture-refactor|2026-05-08 UTK AI Clean Architecture Refactor]]
  - [[agents/sessions/2026-05-09-utk-ai-native-tool-calls|2026-05-09 UTK AI Native Tool Calls]]
  - [[agents/sessions/2026-05-10-utk-ai-chat-log-tool-policy|2026-05-10 UTK AI Chat Log Tool Policy]]

## Validation

- `dotnet build Utk.Ai.slnx`
- `skaffold render --offline=true`
- `skaffold dev --port-forward`
- `dotnet run --project src/Utk.Ai.Api/Utk.Ai.Api.csproj --no-build --urls http://127.0.0.1:5080`
- `cd src/Utk.Ai.Api && dotnet watch run`
- `curl -fsS http://127.0.0.1:5080/health`
- `curl -fsS http://localhost:9596/health`

## Current State

- Repository lives at `/home/marat/dev/git/alfa/utk-ai` and targets `.NET 10` with `Domain`, `Application`, `Infrastructure`, `Api`, `Contracts`, and standalone Blazor WebAssembly `Web` projects.
- Persistence uses PostgreSQL for monitored targets, monitoring runs, aggregation buckets/examples, findings, evidence, investigations, chat conversations/messages, prompt history, and debug entries.
- Local runtime is PostgreSQL through `skaffold dev -p deps --port-forward`, with API and Web run locally by stable `dotnet run`; current preferred health checks are `http://localhost:9596/health` and Web UI smoke/E2E checks.
- `utk-mcp` is the integration boundary for Kibana, Jira, Confluence, Bitbucket, and Jenkins data; local MCP is expected on `http://127.0.0.1:1985/mcp`.
- Monitoring supports raw Kibana sampling and aggregated bucket strategies, stores MCP evidence including `kibanaUrl`, and exposes findings through API and the Blazor UI.
- Monitoring issue investigations and plain Kibana-link investigations are queued, persisted workflows backed by AlfaGen with SignalR progress.
- Chat is the default Web route and uses SignalR plus persisted conversations; it supports native OpenAI chat tool calls for logs and Alfa internal systems.
- Chat-agent orchestration now lives behind Application services, with SignalR hubs acting as transport adapters and tool execution centralized through `IAgentToolExecutor`.
- Clean Architecture boundaries are active: Web depends on `Utk.Ai.Contracts`, public DTO mapping is handled at API/SignalR edges, and Application/Infrastructure do not reference Contracts.
- Recent UI shape is ChatGPT-like chat with optional debug console, service mention picker, prompt history, and VS Code-inspired theme support.
- Console logging is operator-focused: app/AlfaGen progress at `Information`, Microsoft/EF noise at `Warning`, and EF SQL command logs suppressed.
- Detailed chronology for the implementation lives in the related session notes listed above.

## Key Decisions

- Build the app as a `.NET 10` API plus standalone Blazor WebAssembly UI, using PostgreSQL as the durable store.
- Keep local development minimal: Skaffold provides PostgreSQL dependencies, while API and Web run locally with `dotnet run`.
- Use `utk-mcp` as the integration boundary for Alfa internal systems and consume MCP-provided evidence links instead of rebuilding them locally.
- Use the official MCP C# SDK and official OpenAI .NET SDK for the integration layers.
- Treat monitoring dates as UTC and replace stored results only within the selected date/environment/service/strategy scope.
- Store raw aggregation buckets and representative examples separately from AlfaGen findings so MCP grouping remains inspectable.
- Keep full evidence in storage and truncate only prompt payloads sent to AlfaGen.
- Keep monitoring and plain investigations queue-backed, persisted, and SignalR-reported.
- Keep chat transport SignalR-only, with server-side persistence and browser-owned active conversation ID.
- Use native OpenAI chat tool calls for agent loops; keep model tool intent in debug/tool-call events, not assistant transcript markers.
- Keep configured monitoring targets as suggestions, not a hard allowlist, for chat log searches.
- Keep agent tools discoverable through a shared catalog and execute them behind `IAgentToolExecutor`.
- Keep `Utk.Ai.Contracts` as the public HTTP/SignalR DTO boundary; Application and Infrastructure stay free of Contracts references.
- Keep chat orchestration behind `IChatAgentService` and split substantial policies into focused Application collaborators.
- Prefer explicit enum mappings across Domain/Application/Contracts boundaries.
- Use this project note for current durable state and decisions; keep implementation chronology in session notes.

## Blockers

- No current blocker for manual trigger smoke path
