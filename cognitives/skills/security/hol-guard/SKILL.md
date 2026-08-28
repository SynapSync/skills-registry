---
name: hol-guard
description: >-
  Install and operate HOL Guard around local AI harness tool execution.
license: Apache-2.0
metadata:
  author: hashgraph-online
  version: "1.0"
  scope: [root]
  auto_invoke: "protect local AI harnesses with HOL Guard"
allowed-tools: Read, Bash
---

# HOL Guard

## Purpose

Use HOL Guard as a local runtime policy boundary for AI coding harnesses before tools execute. Install Guard, detect the active harness, apply Guard-owned protection, review policy decisions, and collect evidence without replacing the harness's own permissions.

## When to Use This Skill

Use this skill when the user wants to:

- add HOL Guard protection to a local AI coding harness;
- verify whether Guard is installed and active;
- dry-run a Guard-protected harness before normal use;
- review blocked or approval-gated actions;
- collect Guard receipts, inventory, or diagnostic evidence.

## Critical Rules

- Never read `.env` files or expose secrets while troubleshooting.
- Never bypass Guard approvals or claim an action was approved without evidence.
- Do not mark a harness protected until `hol-guard` reports that state.
- Prefer Guard-owned install, repair, and configuration commands over manual harness config edits.
- Detect the active harness dynamically instead of assuming a harness identifier.
- Keep cloud sync optional; local protection must not depend on a cloud connection.

## Install and Detect

Check for the CLI with a cross-platform probe first:

```bash
hol-guard --version
```

If that command is unavailable, prefer an isolated install:

```bash
pipx install hol-guard
```

Then inspect the environment:

```bash
hol-guard status
hol-guard detect --json
```

Use the harness identifier reported by `hol-guard detect --json` in the commands below.

## Protect the Detected Harness

Initialize Guard and let Guard own the harness integration:

```bash
hol-guard bootstrap
hol-guard install <harness>
hol-guard run <harness> --dry-run
```

Review the dry-run output before normal execution. When it is correct:

```bash
hol-guard run <harness>
hol-guard status
hol-guard doctor <harness> --json
```

Do not edit user-level harness configuration directly to imitate what `hol-guard install` does.

## Review Policy Decisions

When Guard blocks or queues an action, inspect the request and evidence first:

```bash
hol-guard approvals
hol-guard approvals open <request-id>
hol-guard receipts
hol-guard diff <harness>
```

Use the pending request identifier returned by `hol-guard approvals`; do not guess or reuse an unrelated request ID.

For terminal-only resolution, use the request identifier returned by Guard:

```bash
hol-guard approvals approve <request-id>
hol-guard approvals deny <request-id>
```

Only approve after understanding the requested scope and risk reason.

## Audit and Diagnostics

Use Guard's evidence surfaces when the user needs proof or troubleshooting data:

```bash
hol-guard receipts
hol-guard inventory
hol-guard abom --format json
hol-guard events
hol-guard doctor
hol-guard detect --json
hol-guard settings show
```

If cloud features are explicitly wanted, inspect connection state rather than assuming it:

```bash
hol-guard connect status
```

## Completion Check

Before reporting success, verify:

1. `hol-guard status` reports the expected protection state.
2. `hol-guard detect --json` resolves the intended harness.
3. A dry run completes without an unexplained bypass.
4. Any pending approvals remain visible rather than being silently overridden.
5. The response states what ran, what Guard found, and what remains blocked or user-controlled.
