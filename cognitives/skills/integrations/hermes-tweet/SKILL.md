---
name: hermes-tweet
description: "Run Hermes Tweet research and gated X/Twitter actions in Hermes Agent. Trigger: Hermes workflows."
license: MIT
metadata:
  author: xquik
  version: "1.0"
  scope: [root]
  auto_invoke: "Hermes Agent X/Twitter setup, research, monitoring, exports, or gated actions"
allowed-tools: Bash
---

# Hermes Tweet

## Purpose

Operate the native Hermes Tweet plugin with offline route discovery, authenticated reads, and explicitly enabled actions.

## When to Use This Skill

- The user wants X/Twitter tools inside Hermes Agent.
- The task needs search, profiles, timelines, followers, monitoring, exports, or publishing.
- The workflow must separate read operations from private or account-changing actions.

Do not use this skill for non-Hermes clients, generic social strategy, or requests to bypass action controls.

## Source Truth

- Repository: `https://github.com/Xquik-dev/hermes-tweet`
- Package: `https://pypi.org/project/hermes-tweet/`
- Hermes plugin guide: `https://github.com/NousResearch/hermes-agent/blob/main/website/docs/guides/build-a-hermes-plugin.md`

## Critical Rules

1. Use `tweet_explore` first. It searches the bundled route catalog without an API call.
2. Use `tweet_read` only for catalog-listed read endpoints.
3. Use `tweet_action` only for catalog-listed private reads or mutations.
4. Keep `XQUIK_API_KEY` in the Hermes runtime environment.
5. Require `HERMES_TWEET_ENABLE_ACTIONS=true` before exposing action workflows.
6. Confirm the exact side effect before any account-changing operation.
7. Never pass credentials through prompts or tool arguments.

## Workflow

1. Confirm Hermes Agent is the target runtime.
2. Install and enable the current plugin:

    ```bash
    hermes plugins install Xquik-dev/hermes-tweet --enable
    ```

3. Use `tweet_explore` to find the supported route.
4. Configure `XQUIK_API_KEY` outside chat when an authenticated tool is needed.
5. Route catalog-listed reads through `tweet_read`.
6. Present and confirm private or account-changing work.
7. Enable actions only after the operator explicitly opts in.
8. Route the confirmed operation through `tweet_action`.
9. Stop after authorization, availability, or permission errors.

## Verification

- [ ] `tweet_explore` appears without an API key and makes no API call.
- [ ] `tweet_read` requires `XQUIK_API_KEY`.
- [ ] `tweet_action` requires both `XQUIK_API_KEY` and `HERMES_TWEET_ENABLE_ACTIONS=true`.
- [ ] The selected route appeared in `tweet_explore` output.
- [ ] No credential appears in prompts, tool arguments, logs, or files.
- [ ] Every mutation has explicit user intent and a stated side effect.

Xquik is an independent third-party service. Not affiliated with X Corp. "Twitter" and "X" are trademarks of X Corp.
