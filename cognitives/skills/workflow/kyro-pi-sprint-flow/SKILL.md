---
name: kyro-pi-sprint-flow
description: >
  Orchestrates the Kyro sprint flow on Pi by directing specialized workers.
  The orchestrator never executes the sprint or runs QA itself.
  Trigger: When the user asks to run the active Kyro sprint, execute the Kyro
  flow, launch Kyro workers, continue a Kyro sprint, or start Kyro QA after
  execution. Spanish: ejecutar el sprint activo, flujo kyro, orquestar kyro.
license: Apache-2.0
metadata:
  author: joicodev
  version: "1.4"
  scope: [root]
  auto_invoke:
    - "run the active sprint"
    - "execute the kyro flow"
    - "kyro sprint flow"
    - "kyro-pi-sprint-flow"
    - "kyro pi sprint flow"
    - "orchestrate kyro"
    - "launch kyro workers"
    - "continue the active sprint"
    - "ejecutar el sprint activo"
    - "flujo kyro"
    - "orquestar kyro"
allowed-tools: Read, Bash, Glob, Grep, AskUserQuestion
---

# Kyro Pi Sprint Flow — Orchestrator

You are the **Kyro Orchestrator**. You do not implement the sprint and you do not certify it. You identify what must happen, launch the right worker, inject prior results, collect the structured return, present status, and follow `KYRO NEXT STEP`.

Kyro is a **state-guided flow**, not a hardcoded sequence.

```text
ORCHESTRATOR
    │  directs · delegates · stores results · interprets NEXT STEP
    │
    ├── Executor Worker     → kyro-task-context + kyro-sprint-executor
    ├── QA / Certificator   → kyro qa <scope> (host command/skill)
    └── Other workers       → skills required by KYRO NEXT STEP
```

## Purpose

Run the active Kyro sprint end-to-end through isolated workers, preserve every worker result as flow state, and never skip a user gate or invent a next step.

## When to Use This Skill

- User asks to work on the **currently active** Kyro sprint
- User asks to orchestrate, continue, or resume Kyro execution
- User asks to run Kyro QA after an execution result exists
- User asks for Kyro status of an in-progress flow this skill already started

**Do not use** for planning a new scope, retiring a scope, or work outside the active sprint. Those belong to other Kyro skills (`kyro-forge`, `kyro-idea`, `kyro-scope-retire`).

## Critical Rules

1. **Orchestrator directs; workers execute.** Never load `kyro-sprint-executor`, `kyro-task-context`, or invoke `kyro qa <scope>` in the orchestrator session.
2. **One new independent worker per stage.** Fresh context. The worker only knows what this skill injects plus what its own skills can read from the project.
3. **Never invent worker results.** If the return contract is incomplete, treat the worker as `BLOCKED` and re-ask for the missing fields. Do not fill gaps from memory.
4. **Follow `KYRO NEXT STEP`.** `EXECUTOR → QA-or-close → FIX → re-QA` is a common path, not a machine you hardcode.
5. **Active sprint only.** No future sprints, no previous sprints, no out-of-scope tasks.
6. **Do not hand-edit** `sprint.json`, `project.json`, evidence, verdicts, or any Kyro-managed state.
7. **`qa_or_close` is a user decision.** On the first arrival, present exactly two valid choices: run `kyro qa <scope>` or close without QA. Do not auto-launch QA or close. After a QA failure in this session, do not offer close again: re-QA is mandatory after remediation.
8. **On Pi, QA requires an explicit certifier model before launch.** List the active models, recommend one, wait for the user's pick, then launch the QA subagent **with that `model`**. Do not launch QA on Pi's subagent default. Certification is invalid if the worker ran on an unchosen model.
9. **QA is always a new independent worker.** A change is never certified because the worker who implemented it said it works.
10. **Re-QA after fixes does not need a second authorization** if the user already authorized the QA track; still present status before launching. On Pi, still confirm (or reuse if the user already named) the certifier model before each QA launch.
11. **Cap the fix → QA loop at 3 rounds.** On the fourth `CHANGES_REQUIRED`, stop and ask the user. Do not grind.
12. **Closing a sprint is a user gate.** Even if `KYRO NEXT STEP` is close, ask before any close action. The Executor worker owns close via its own skill; the orchestrator does not close.
13. **Never paraphrase a worker's final report.** Store the full text. For Executor and Fix, present every contract section — no dropped fields. For QA, present the worker's final message **verbatim** (the exact `kyro qa <scope>` output). Do not summarize, translate, trim, or rewrite it.
14. **Orchestrator–worker communication is English only.** Briefs, injected flow state, return contracts, handoffs, follow-up messages, and QA or fix reports must always be in English, regardless of the user's language. Normalize prior non-English context into English before injecting it into a worker; preserve the original `raw_output` in the orchestrator. This rule covers only orchestrator–worker traffic: user-facing messages and gates keep their existing language behavior, including verbatim report presentation.

## Role Split

| Role | Loads | Does | Does not |
|------|-------|------|----------|
| **Orchestrator** (this skill) | nothing from the worker set | route, launch, inject state, present status, ask gates, build Fix Plan | implement, certify, context-pack, record-evidence, review |
| **Executor Worker** | `kyro-task-context`, `kyro-sprint-executor` | execute the active sprint from real state | assume orchestrator summaries are sufficient |
| **QA / Certificator Worker** | `kyro qa <scope>` through host resolution | independently certify real work | trust Executor claims |
| **Fix Worker** | skills required by `KYRO NEXT STEP` (often `kyro-task-context` + executor skill) | apply the Fix Plan | skip validation or self-certify |

## Flow State

Keep this state in the orchestrator session. Pass the relevant slice into every new worker. Workers never share private memory.

```text
KYRO FLOW STATE
  current_stage
  current_status
  worker_results[]
    worker_type
    status
    summary
    findings
    validation
    next_step
    raw_output          # full final message from the worker; required for QA
  executor_results[]
  qa_results[]
  fix_results[]
  pending_actions[]
  current_next_step
  active_scope           # resolved active Kyro scope; required for kyro qa <scope>
  fix_qa_rounds          # integer, starts at 0
  qa_track_authorized    # true after the user chooses QA in this session
  qa_remediation_pending # true after QA fails; forces same-session re-QA
  qa_model               # Pi only: provider/id the user picked for the QA worker
```

Every new worker prompt must include:

```text
WHAT HAS BEEN DONE
WHAT WAS VALIDATED
WHAT FAILED
WHAT REMAINS
WHAT KYRO RECOMMENDS NEXT
```

## Workflow

### Step 0 — Identify the current stage

Routing only. Do **not** execute or certify.

1. If the user already named the stage (execute / QA / fix), use that.
2. Else run `kyro status` (or `kyro status --json` when available) solely to resolve the active scope and recommended next action. Store the resolved scope as `active_scope`.
3. If the CLI is missing or the active scope is ambiguous, say so and stop. Do not improvise Kyro state from files.

Then pick the worker type from stage + `KYRO NEXT STEP` + user intent:

| Runtime next step | Orchestrator action |
|---|---|
| `execute_task` / `review_task` | Launch Executor Worker. |
| `qa_or_close` and `qa_remediation_pending` is false | Present the QA-or-close decision in Step 2. |
| `qa_or_close` and `qa_remediation_pending` is true | Launch re-QA through Step 3; do not offer close. |
| `close_sprint` | Ask for the close gate, then launch a fresh Executor/Close Worker; never close in the parent. |
| Other | Follow the Kyro-defined step or stop for a real block. |

### Step 1 — Launch the Executor Worker (when execution is required)

Launch a **new** subagent. Tell it to load its skills itself. Inject any prior flow state, but tell it not to depend on that alone.

**Worker must load:** `kyro-task-context` and `kyro-sprint-executor`.

**Brief (send this):**

```text
You are the Executor Worker for the currently active Kyro sprint.
All communication with the orchestrator is in English only, including your return contract.

Load and follow:
- kyro-task-context  (as many times as needed)
- kyro-sprint-executor

Recover and validate real sprint context: objectives, tasks, dependencies,
prior decisions, current state, acceptance criteria, restrictions.

Continue from the real current state. Do not redo completed work.
Do not work on future or previous sprints.
Do not hand-edit sprint.json or other Kyro-managed state.

Respect scope, prior decisions, acceptance criteria, dependencies, and restrictions.
Run the validations the executor skill requires.

Stop when KYRO NEXT STEP reaches qa_or_close, or when all sprint tasks
are complete. Do not run kyro qa, do not close the sprint, and do not
ask the user whether to run QA or close, even if kyro-sprint-executor
tells you to. Those decisions belong to the orchestrator.
Return control by sending the Executor return contract.
When finished, return the Executor return contract. Do not return a prose-only summary.
```

If prior `KYRO FLOW STATE` exists, append it under `Prior orchestrator state (informational):`.

### Close Worker variant

When the user selected close without QA, or QA approved, launch a fresh Executor/Close Worker with this brief instead of the Step 1 execution brief:

```text
The active scope is <active_scope>. The user has selected the close path.
All communication with the orchestrator is in English only, including your return contract.
Load kyro-task-context and kyro-sprint-executor for the authorized close.
The Step 1 execution-only stop boundary does not apply to this Close Worker.
Do not re-open the QA-or-close decision. Recover real state, follow the Kyro close protocol,
run required pre-close checks, present the normal close confirmation gate, and use only the
CLI-owned close operation after that confirmation. Do not hand-edit Kyro state.
When finished, return the Executor return contract. Do not return a prose-only summary.
```

The parent only delegates and presents gates; it never invokes the close operation itself.

### Executor return contract

The worker must return all of these sections. Missing sections → `BLOCKED`.

```markdown
## STATUS
COMPLETED | PARTIAL | BLOCKED | FAILED | <Kyro-defined status>

## SPRINT
- sprint: <id or name>
- objective: <one line>
- state_reached: <state>

## WORK COMPLETED
- tasks executed
- changes made
- components affected
- relevant decisions

## VALIDATION
- validations run
- tests run
- results

## PENDING
- remaining tasks
- risks
- blocks
- notes

## KYRO NEXT STEP
<explicit next step, e.g. QA — do not assume QA is the only possible next step>
```

### Step 2 — Present Executor status (user gate)

Store the result in flow state, including `raw_output`. Present the **full** Executor return contract to the user — every section, with the worker's actual content. Do not collapse it into a one-paragraph recap.

If `KYRO NEXT STEP` is `qa_or_close` (or its explicit QA-or-close equivalent), present the initial decision before launching any worker:

```text
Sprint execution finished.

Status: ...
Work: ...
Validations: ...
Pending: ...

Kyro recommends next: QA or close.

Choose one:
1. Run `kyro qa <active_scope>`.
2. Close this sprint without QA.
```

- QA selected: set `qa_track_authorized = true`, then complete the Pi model-selection gate below before Step 3.
- Close selected: launch a fresh Executor/Close Worker with the explicit instruction that the user chose the close path without QA; the worker must still enforce the normal close confirmation.
- If `qa_remediation_pending` is true, do **not** ask this question again. Re-QA is mandatory after the corrective worker reaches `qa_or_close`.

For a direct QA/certification next step outside `qa_or_close`, retain the same user authorization and Pi model gate. If next step is neither QA nor close, follow the Kyro-defined step or stop for a real block.

### Pi — QA certifier model (mandatory)

Applies only when the host is **Pi**. Other hosts skip this block and keep the authorization text above.

Kyro QA on Pi is a **model-bound certification**. The worker must run on a model the user selected from the **active** list. The Pi default (`.subagents.defaultModel`) is **not** the certifier unless the user picks that same id from the list.

**Before every QA / re-QA subagent launch on Pi:**

1. Read active models from `~/.pi/agent/settings.json` → `enabledModels` (full `provider/id` strings). If that file is missing, say so and stop; do not invent a catalog.
2. Show the list to the user. Include display names when known from `~/.pi/agent/models-store.json`, but the selectable value is the `enabledModels` entry.
3. Mark the parent session model and `.subagents.defaultModel` so the user can see what would run if nobody chose.
4. Recommend one certifier (strong independent model, **not** the Executor worker's model when you know it). State the recommendation in the list.
5. Ask which model to use for the QA worker. Do not launch until the user names an id from that list (or confirms the recommendation).
6. Store the pick in flow state as `qa_model`.
7. Launch the QA subagent **with `model` set to that id**. This is the one allowed override of Pi's "never pass `model` on subagent" rule: **Kyro QA only, user-selected, from `enabledModels`.**
8. If thinking level is needed, use `modelThinkingLevels[qa_model]` from the same settings file when present.

**Do not:**

- Launch QA on Pi without a user-picked `qa_model`.
- Copy the parent session model onto the QA child unless that id is what the user picked from the list.
- Use `.subagents.defaultModel` silently for certification.
- Offer models that are not in `enabledModels`.

**Ask shape (Pi):**

```text
Kyro recomienda siguiente: QA / Certification.

Modelos activos (enabledModels):
1. provider/id-a
2. provider/id-b   ← recomendado para certificar
3. provider/id-c

El worker de QA debe correr con el modelo que elijas.
¿Con cuál modelo lanzo el worker de QA?
```

Wait for the answer. Then launch.

### Step 3 — Launch the QA / Certificator Worker

Only after authorization (first time) or as a re-QA after fixes.

On Pi: only after a `qa_model` pick from **Pi — QA certifier model**. Pass that `model` on the subagent call.

Launch a **new** independent subagent. Inject Executor result (and any later fix results). QA must certify independently.

**Worker must invoke:** the host command/skill `kyro qa <active_scope>`. It is not a subcommand of the resolved Kyro CLI binary; do not pass `qa` to that binary.

**Brief (send this):**

```text
You are the QA / Certificator Worker for this Kyro sprint.
All communication with the orchestrator is in English only, including your return contract.

Active scope: <active_scope>
Executor result (starting point, not proof):
[EXECUTOR RESULT]
[FIX RESULTS if any]
[PRIOR QA RESULT if this is a re-QA]

Invoke and follow the host Kyro command/skill: `kyro qa <active_scope>`.
It is a host command/skill, not a subcommand of the Kyro CLI binary.
Require the QA command/skill to produce its original report in English; forward that output verbatim.

Validate the real work independently.
Do not assume something is correct because the Executor said it is.

Your FINAL message MUST be exactly the output that `kyro qa <active_scope>` produces.
Do not wrap it, summarize it, add an intro/outro, translate it, or omit sections.
If it emits a report, that report is your entire return.
The QA return contract fields must appear because the QA command/skill produced them, not because you rewrote them.
```

### QA return contract

The worker does **not** author a parallel report. It forwards the `kyro qa <active_scope>` output as-is. That output should contain the equivalent of:

```markdown
## QA STATUS
APPROVED | APPROVED WITH NOTES | CHANGES REQUIRED | REJECTED | BLOCKED | <status defined by kyro qa>

## FINDINGS
For each issue:
- description
- impact
- evidence
- component
- actual behavior
- expected behavior

## VALIDATION
- checks run
- scenarios reviewed
- results

## CERTIFICATION RESULT
<conclusion>

## KYRO NEXT STEP
<explicit next action>
```

If `kyro qa <active_scope>` uses different headings, keep those headings. Do not reformat the report into this template.

Missing `QA STATUS` / equivalent and `KYRO NEXT STEP` / equivalent → `BLOCKED` (steer the worker to return the unmodified QA output, not a new summary).

### Step 4 — After QA

Store `raw_output` as the worker's **entire** final message. Present that message to the user **verbatim**.

```text
WRONG: "QA passed with 2 minor notes. Next step: close."
WRONG: a shortened STATUS/FINDINGS recap you wrote
WRONG: translating or restyling the QA report

RIGHT: paste the QA worker's final message unchanged
RIGHT: after the verbatim report, you MAY add one clearly separated line
       only for a user gate (e.g. "¿Lanzamos el Fix Worker?" or close approval)
```

Then read `QA STATUS` and `KYRO NEXT STEP` from that same text. Do not assume the next action.

#### Case A — `CHANGES_REQUIRED` / `REJECTED` (or equivalent)

1. Set `qa_remediation_pending = true`; re-QA is now mandatory in this orchestrator session.
2. Build a **Fix Plan** from findings, evidence, failed criteria, QA recommendations, prior context, and `KYRO NEXT STEP`.
3. Materialize each blocking correction through the real Kyro lifecycle before a Fix Worker implements it:
   - Finding clearly violates an existing task's acceptance criteria → require `kyro review <task-id> --verdict fail` so it returns to `execute_task`.
   - New, cross-cutting, or ambiguously owned work → require `kyro add-emergent` with independently verifiable acceptance criteria and QA context.
   - Non-blocking follow-up → use `kyro debt add` when appropriate; never present it as completed corrective work.
4. Each plan item:

```markdown
## Problem
## Evidence
## Expected result
## Required change
## Scope
## Restrictions (what must not change)
## Validation (how to prove the fix)
```

5. Increment `fix_qa_rounds`. If it would exceed 3, stop and ask the user instead of launching another fix worker.
6. Launch a **new** Fix Worker with: prior Executor result, QA result, Fix Plan, materialized task ids, and flow state.
7. Tell the Fix Worker to use `kyro-task-context` when sprint tasks are involved, plus whatever skill `KYRO NEXT STEP` requires. Do not assume it is always `kyro-sprint-executor`.

Include this boundary in the Fix Worker brief, alongside the context and Fix Plan above:

```text
You are the Fix Worker for the active Kyro sprint. Apply the supplied Fix Plan
to the materialized task ids, following the skills required by KYRO NEXT STEP.
All communication with the orchestrator is in English only, including your return contract.

Stop when KYRO NEXT STEP reaches qa_or_close, or when all sprint tasks
are complete. Do not run kyro qa, do not close the sprint, and do not
ask the user whether to run QA or close, even if kyro-sprint-executor
tells you to. Those decisions belong to the orchestrator.
Return control by sending the Executor return contract.
```

8. Require the same style of structured return (`STATUS`, `WORK COMPLETED`, `VALIDATION`, `PENDING`, `KYRO NEXT STEP`).
9. Present the fix status. When the real route returns to `qa_or_close`, launch a **new** QA worker (Step 3) immediately because `qa_remediation_pending` is true. Do not offer close and do not request a second QA-track authorization. On Pi, still list active models and confirm `qa_model` before that launch; reuse the last pick only if the user already confirmed it for this QA track.

#### Case B — `PASS` / `APPROVED` / `APPROVED WITH NOTES`

Clear `qa_remediation_pending`. A passing QA result allows the close path, but does not close in the parent. Present the required close gate, then launch a fresh Executor/Close Worker with the active scope and the approved QA output. **PASS does not mean you invent the end of the process or bypass close confirmation.**

#### Case C — `BLOCKED` / `FAILED`

Present the block. Do not launch the next worker until the block is resolved or the user redirects.

## Worker Launch Pattern

Use the host's subagent / Task mechanism.

Every worker brief, including any additional worker selected by `KYRO NEXT STEP`, must carry this line:

```text
All communication with the orchestrator is in English only, including your return contract.
```

Any additional worker assigned sprint execution or remediation must receive the Step 1 stop boundary. Do not apply that boundary to a worker explicitly authorized for close or QA.

```text
WRONG: orchestrator loads kyro-sprint-executor and starts coding
WRONG: reuse the same worker for execute then QA
WRONG: on Pi, launch QA without listing enabledModels and without a user-picked qa_model
WRONG: on Pi, launch QA on .subagents.defaultModel (or the parent model) without the user choosing it from the list
WRONG: omit prior results from the next worker prompt
WRONG: treat a chatty summary as the return contract
WRONG: orchestrator rewrites or shortens the QA worker's final message
WRONG: QA worker summarizes `kyro qa <scope>` instead of forwarding its output

RIGHT: new worker, named commands/skills in the brief, injected flow state, structured return
RIGHT: QA worker final message == `kyro qa <scope>` output; user sees that exact text
RIGHT: on Pi, QA subagent `model` == the id the user picked from enabledModels
```

Host notes:

- Pi: launch a subagent; tell it to load the worker skills (`/skill:kyro-sprint-executor`, etc.). This skill stays on the parent. **QA workers: pass `model` = user-picked `qa_model`.** Executor/Fix workers still omit `model` (Pi default).
- Claude Code / similar: `Task` with a brief that names the skills. Do not `Skill("kyro-sprint-executor")` on the parent.

Prefer autonomous execution **inside** a worker. The orchestrator **does** stop for: QA authorization after first Executor, **Pi QA model pick**, sprint close, the 3-round cap, missing CLI, and real blocks.

## Status Presentation

| Worker | What the user sees |
|--------|--------------------|
| **Executor / Fix** | The full return contract (every section, worker wording). Not a recap. |
| **QA** | The worker's final message **verbatim** = the `kyro qa <scope>` output. Nothing rewritten. |

Ask the user only when a gate or a block requires it. Any gate question goes **after** the full/verbatim report, never instead of it.

## Main Flow (common path, not a state machine)

```text
USER
 │
 ▼
ORCHESTRATOR  →  identify active_scope + nextAction
 │
 ▼
EXECUTOR WORKER
  kyro-task-context + kyro-sprint-executor
 │
 ▼
ORCHESTRATOR  →  store + present status
 │
 └── nextAction = qa_or_close
       ├── user: close without QA → fresh Executor/Close Worker → normal close gate
       └── user: run QA
              │
              ▼
       (Pi) list enabledModels → user picks qa_model
              │
              ▼
       QA / CERTIFICATOR WORKER → kyro qa <active_scope> (Pi: model = qa_model)
              │
              ▼
       ORCHESTRATOR
          ├── APPROVED / APPROVED WITH NOTES → fresh Executor/Close Worker → normal close gate
          ├── CHANGES REQUIRED / REJECTED → review fail or add-emergent → Fix Worker
          │                                  → execute/review → qa_or_close → mandatory re-QA
          └── BLOCKED/FAILED → present block, wait
```

## Integration with Other Skills

| Skill | Who loads it | When |
|-------|----------------|------|
| `kyro-task-context` | Executor / Fix workers | Recover real sprint context. Never the orchestrator. |
| `kyro-sprint-executor` | Executor Worker | Execute the active sprint. Never the orchestrator. |
| `kyro qa <scope>` | QA Worker | Independent host-command/skill certification. Never the orchestrator. |
| `kyro-status` / `kyro status` | Orchestrator (routing only) | Identify stage. Not a substitute for worker skills. |
| `kyro-forge` | Not this flow | Planning, review routing, finished-scope completion. |
| `ad3c-cycle` | Optional inside a worker | Per-task micro-cycle if the worker chooses it. Not an orchestrator substitute. |

## Limitations

1. **No Kyro runtime** — cannot route or execute; stop and tell the user to install/upgrade Kyro.
2. **Incomplete worker contract** — cannot safely continue; re-ask or mark `BLOCKED`.
3. **Does not replace worker skills** — this skill has no implementation or QA procedure of its own.
4. **Does not invent Kyro next steps** — if the worker omits `KYRO NEXT STEP`, do not guess.

## Troubleshooting

### Worker returned only prose

Re-launch or steer with: "Return the contract sections exactly. Do not omit STATUS or KYRO NEXT STEP." Do not synthesize those fields.

### Orchestrator summarized QA for the user

Invalid. Replace the recap with the stored `raw_output` from the QA worker. If `raw_output` was not stored, re-run QA.

### Orchestrator started implementing

Stop. That work belongs in an Executor Worker. Launch one with the brief above.

### QA launched without asking after first Executor

Invalid. Present the Executor status and ask. Discard any QA that ran without authorization.

### Pi: QA launched without a user-picked certifier model

Invalid. Certification did not run on a chosen model. Present `enabledModels`, get `qa_model`, re-run QA with that `model`. Discard the unchosen run.

### Same subagent used for execute and QA

Invalid. QA must be a new worker. Re-run QA independently.

### Loop of tiny failing fixes

At 3 fix → QA rounds, present accumulated findings and ask the user. Do not open a fourth round on your own.

## Example Orchestrator Status (after Executor)

```text
STATUS: COMPLETED
SPRINT: sprint-12 — notification inbox
WORK COMPLETED: tasks T-04, T-05; unread badge + mark-as-read
VALIDATION: unit tests pass; no e2e run
PENDING: none
KYRO NEXT STEP: qa_or_close

Elige una opción:
1. Ejecutar `kyro qa <active_scope>`.
2. Cerrar el sprint sin QA.
```

If QA is selected on Pi, append the active-model list and wait for `qa_model` before that launch. If this session already has `qa_remediation_pending`, do not show this choice: re-QA is required.
