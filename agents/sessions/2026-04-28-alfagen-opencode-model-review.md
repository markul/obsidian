---
tags:
  - agents/session
note-type: agent-session
session-date: 2026-04-28
related-project:
related-ticket:
---

# 2026-04-28 alfagen-opencode-model-review

## Goal

- Review which AlfaGen models are currently available to local `opencode`

## Scope

- Local config: `/home/marat/.config/opencode/opencode.json`
- Provider base URL from config: `https://alfagen.moscow.alfaintra.net/continue-dev`
- User-provided URL variant: `https://alfagen.moscow.alfaintra.net/continue-de`

## Actions

- Read the local `opencode` config and redacted the provider details in-memory to avoid exposing the stored API key
- Confirmed the configured provider name is `alfagen` and the configured default model is `alfagen/zai-org/GLM-5-FP8`
- Queried the live provider `GET /models` endpoint against the configured `continue-dev` base path
- Sanity-checked both `continue-de` and `continue-dev` path variants at the root and `/models`
- Compared the live advertised model list with the locally hardcoded `provider.alfagen.models` list in `opencode.json`
- Sent lightweight `POST /chat/completions` probes with tiny token limits to distinguish models that accept requests from models that are absent or slow
- Updated `/home/marat/.config/opencode/opencode.json` to match the live `/models` list: added `MiniMaxAI/MiniMax`, removed stale `zai-org/GLM-5-FP8` and `Qwen/Qwen3-235B-A22B-Instruct-2507-FP8`, changed the default model to `alfagen/Qwen/Qwen3-Coder-30B-A3B-Instruct`, and then switched it again to `alfagen/MiniMaxAI/MiniMax` on request

## Findings

- The working OpenAI-compatible discovery endpoint is `https://alfagen.moscow.alfaintra.net/continue-dev/models`
- Both root URLs returned `500`, and `continue-de/models` also returned `500`
- The live `/models` response advertised exactly these six models:
  - `Qwen/Qwen3-Coder-Next`
  - `MiniMaxAI/MiniMax`
  - `Qwen/Qwen3.5-397B-A17B-FP8`
  - `Qwen/Qwen3-Coder-30B-A3B-Instruct`
  - `moonshotai/Kimi-K2-Thinking`
  - `openai/gpt-oss-120b`
- The local `opencode` config currently lists seven models, but it is out of sync with the provider:
  - present in both config and live `/models`: `Qwen/Qwen3-Coder-Next`, `Qwen/Qwen3-Coder-30B-A3B-Instruct`, `Qwen/Qwen3.5-397B-A17B-FP8`, `moonshotai/Kimi-K2-Thinking`, `openai/gpt-oss-120b`
  - present live but missing from config: `MiniMaxAI/MiniMax`
  - present in config but missing from live `/models`: `zai-org/GLM-5-FP8`, `Qwen/Qwen3-235B-A22B-Instruct-2507-FP8`
- Minimal completion probes returned `200` for:
  - `Qwen/Qwen3.5-397B-A17B-FP8`
  - `Qwen/Qwen3-Coder-30B-A3B-Instruct`
  - `MiniMaxAI/MiniMax`
  - `moonshotai/Kimi-K2-Thinking`
  - `openai/gpt-oss-120b`
- The three thinking-style models above returned only `reasoning_content` under the tiny token budget, which still confirms the endpoint accepted the model and generated output
- `Qwen/Qwen3-Coder-Next`, `zai-org/GLM-5-FP8`, and `Qwen/Qwen3-235B-A22B-Instruct-2507-FP8` timed out under the lightweight probe, so they should be treated as unverified for interactive `opencode` use from this workstation

## Decisions

- Treat live `/models` output as the source of truth for what AlfaGen currently exposes
- Treat the local default model `alfagen/zai-org/GLM-5-FP8` as suspect until it is either restored to `/models` or manually revalidated with a longer-running request
- Use `alfagen/MiniMaxAI/MiniMax` as the current local default because it is live and explicitly requested as the default

## Validation

- `python3` against `/home/marat/.config/opencode/opencode.json`
- `GET https://alfagen.moscow.alfaintra.net/continue-dev/models`
- lightweight `POST https://alfagen.moscow.alfaintra.net/continue-dev/chat/completions` probes per model

## Outcome

- The local `opencode` AlfaGen provider is configured and reachable, but its configured model list is stale relative to the live provider
- The safest currently confirmed interactive choices are `Qwen/Qwen3.5-397B-A17B-FP8` and `Qwen/Qwen3-Coder-30B-A3B-Instruct`
- The local `opencode` config now matches the live AlfaGen model list and no longer references the two stale/missing models
- The current local default model is now `alfagen/MiniMaxAI/MiniMax`
