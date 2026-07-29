---
name: kyro-sprint-executor
description: >
  Manual-only. Strict Kyro sprint execution with mandatory per-task
  workflow: context-pack, implement, validate, record-evidence, review.
  Invoke explicitly when executing a Kyro sprint.
license: Apache-2.0
metadata:
  author: joicodev
  version: "1.0"
  scope: [root]
allowed-tools: Bash, Read, Write
---

# Kyro Sprint Executor — Strict Sprint Execution

Execute **Sprint X** exclusively through Kyro's official router and commands.

## Mandatory Workflow for Each Task

Process the sprint tasks in the order defined by Kyro. For every task, execute the following sequence:

1. `context-pack`
2. Implementation
3. Validation
4. `record-evidence`
5. `review`

Do not skip, reorder, or run these stages in parallel.

## Advancement Rule

A task may be marked as completed, and execution may proceed to the next task, only when:

* All required validations have completed successfully.
* Evidence has been registered through `record-evidence`.
* The `review` process has produced `verdict: pass`.
* There are no unresolved errors, critical findings, pending validations, or incomplete evidence.

### Failure Recovery

When a task receives `verdict: fail`:

1. Stop sprint execution immediately.
2. Analyze and document the root cause.
3. Apply only the changes required to resolve the failure.
4. Repeat validation, `record-evidence`, and `review`.
5. Continue only after obtaining `verdict: pass`.

Never force, fabricate, manually override, or bypass a verdict.

## Sprint Closure

After every task has received `verdict: pass`:

1. Verify that there are no open tasks, missing evidence, unresolved findings, or pending validations.
2. Execute the official `close-sprint` command.
3. Confirm that Kyro successfully closed the sprint.

## Critical Restrictions

* Never modify `sprint.json` manually.
* Never directly alter Kyro-managed state or artifacts.
* Never mark a task as completed without verifiable evidence.
* Never close a partially completed sprint.
* Never continue after a failed validation or failed review.
* Never replace official Kyro commands with manual file changes.
* If a Kyro command fails, stop execution, diagnose the issue, correct it through the supported workflow, and rerun the command.

## Expected Final Report

At the end of the execution, provide a concise report containing:

* Tasks executed.
* Validation results.
* Evidence recorded.
* Final verdict for each task.
* Corrections made after failed validations or reviews.
* Result of the `close-sprint` command.
* Confirmation that `sprint.json` was not modified manually.
