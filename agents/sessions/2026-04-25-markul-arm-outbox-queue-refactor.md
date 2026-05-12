---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-25
service: "[[personal/services/markul.arm|markul.arm]]"
project:
ticket:
---

# markul.arm Outbox Queue Refactor

## Goal

- Replace the awkward `EventWaitHandleExtensions`-based async wait path in the Kafka outbox queue with a simpler async-native design

## Scope

- Repository: `/home/marat/dev/git/markul/Markul.Arm`
- Area: `src/Infrastructure/Markul.Kafka/Outbox`

## Actions

- Reviewed `EventWaitHandleExtensions` and its only consumer in `ThreadSafeQueue`
- Reworked `ThreadSafeQueue` around `Channel<long>` batch consumption semantics
- Removed the old extension entirely
- Added queue-focused unit tests and exposed `Markul.Kafka` internals to `Markul.Tests`
- Ran targeted tests for `Markul.Tests` and `Markul.Media.Tests`

## Decisions

- Keep the queue semantics as “enqueue once, dequeue a processing batch, release ids after processing” and model that explicitly instead of splitting it across `WaitDataAsync`, `PeekAll`, and `DequeueBatch`
- Use `Channel<long>` for async waiting and batch draining, while retaining a pending-id snapshot so the retry service can avoid re-enqueuing currently queued or in-flight rows
- Prefer testing the queue directly over preserving the old extension as an independently tested utility

## Validation

- `dotnet test src/Tests/Markul.Tests/Markul.Tests.csproj -c Release --no-restore`
- `dotnet test src/Tests/Markul.Media.Tests/Markul.Media.Tests.csproj -c Release --no-restore`

## Outcome

- The outbox queue now uses channel-backed batch consumption with explicit batch completion instead of the old `EventWaitHandle` bridge, with targeted tests covering enqueue, cancellation, and pending-id lifecycle
