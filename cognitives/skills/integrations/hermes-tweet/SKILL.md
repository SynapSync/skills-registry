---
name: hermes-tweet
description: "Hermes Agent X/Twitter plugin guidance with gated read and action workflows. Trigger: When an AI assistant needs Hermes Agent to research, read, or prepare approved actions for X/Twitter."
license: MIT
metadata:
  author: Xquik
  version: "0.1.6"
  scope: universal
  auto_invoke: "When using Hermes Agent for X/Twitter research, reads, or gated actions"
allowed-tools: []
---

# Hermes Tweet

## Purpose

Guide AI assistants that use Hermes Agent with the Hermes Tweet plugin for X/Twitter discovery, read workflows, and user-approved actions.

## When to Use This Skill

- The user asks Hermes Agent to search, inspect, summarize, or prepare X/Twitter work.
- The workspace has installed the canonical plugin from https://github.com/Xquik-dev/hermes-tweet.
- The task needs safe boundaries between discovery, credentialed reads, and account-changing actions.

## Critical Rules

1. Use `tweet_explore` for credential-free discovery, planning, and examples.
2. Use `tweet_read` only when `XQUIK_API_KEY` is configured.
3. Use `tweet_action` only when `XQUIK_API_KEY` is configured and `HERMES_TWEET_ENABLE_ACTIONS` is set to `true`.
4. Ask for explicit user confirmation before posting, liking, following, reposting, deleting, or changing account state.
5. Never expose API keys, cookies, private account identifiers, raw runtime logs, or hidden implementation details.
6. Keep action payloads reviewable before execution and do not retry write actions after authentication or permission errors.

## Workflow

1. Confirm the user's goal and separate research from account-changing action.
2. Use `tweet_explore` first when the task can be planned without credentials.
3. Use `tweet_read` for live read operations after confirming credentials are configured.
4. Present a concise summary with relevant source URLs or identifiers.
5. For actions, show the exact intended action and wait for user approval.
6. Use `tweet_action` only after approval and only when the action gate is enabled.

## Validation

- Canonical source: https://github.com/Xquik-dev/hermes-tweet
- Plugin version: 0.1.6
- License: MIT
- Expected gates: `tweet_explore` credential-free, `tweet_read` API-key gated, `tweet_action` API-key and explicit action-enable gated.
