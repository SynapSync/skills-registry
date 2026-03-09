---
name: universal-planner
description: >
  Unified planning and execution skill for any software scenario. PLAN mode produces
  structured documentation; EXECUTE mode implements sprints from plan output.
  Trigger: When planning or executing any software work that requires structured analysis and actionable task plans.
license: Apache-2.0
metadata:
  author: synapsync
  version: "3.6"
  scope: [root]
  auto_invoke:
    - "Planning a new project, feature, refactor, or any software work"
    - "Generate project planning documentation"
    - "Create requirements, design, and execution plans"
    - "Analyze and plan a bug fix, tech debt reduction, or architecture change"
    - "Execute the project plan"
    - "Start implementing the sprints"
    - "Work on the next sprint from the planning"
    - "Continue executing the plan"
allowed-tools: Read, Edit, Write, Glob, Grep, Bash, Task
---

# Universal Planner

## Assets

This skill uses a modular assets architecture. Detailed documentation, templates, and helpers are in the [assets/](assets/) directory:

- **[assets/modes/](assets/modes/)** — PLAN mode, EXECUTE mode, and 6 planning sub-modes
- **[assets/helpers/](assets/helpers/)** — Config resolution, decision log, code quality, troubleshooting
- **[assets/templates/](assets/templates/)** — Document templates with frontmatter
- **[assets/validators/](assets/validators/)** — Output validation schemas
- **[assets/examples/](assets/examples/)** — Test cases and examples

See [assets/README.md](assets/README.md) for full directory documentation.

---

## Purpose

Unified planning and execution skill for **any** software engineering scenario. This skill operates in two mutually exclusive modes:

- **PLAN mode** — Produce professional, structured planning documentation. Detects the scenario (new project, feature, refactor, bug fix, tech debt, architecture) and activates the appropriate sub-mode. **Never executes code.**
- **EXECUTE mode** — Consume plan output and implement it sprint-by-sprint, phase-by-phase, task-by-task using dual roles (Senior Developer + Scrum Master). **Never creates plans.**

---

## Critical Rules

> **RULE 1 — MODE BOUNDARY: PLAN NEVER EXECUTES, EXECUTE NEVER PLANS**
>
> PLAN mode produces **documentation only** — it MUST NOT implement, code, build, or deploy. EXECUTE mode implements **existing plans only** — it MUST NOT create new planning documents, modify strategy, or restructure sprints. This boundary is absolute.

> **RULE 2 — CONTEXT-FIRST: RESPECT EXISTING PATTERNS**
>
> Before making any decision on an existing codebase, explore and understand the project's established patterns, conventions, and architecture. Every proposal and implementation MUST align with what already exists. Deviations must be documented and justified.

> **RULE 3 — ADAPTIVE MODE DETECTION**
>
> Detect the top-level mode (PLAN vs EXECUTE) and planning sub-mode from the user's input. If ambiguous, ask. Never force a mode that doesn't match the scenario.

> **RULE 4 — PROFESSIONAL QUALITY**
>
> In PLAN mode: output must read as if produced by a senior engineering team — no vague language, no placeholders. In EXECUTE mode: write production-quality code — clean, readable, properly tested.

> **RULE 5 — COMPLETENESS**
>
> PLAN mode: generate ALL documents required by the active sub-mode. EXECUTE mode: complete ALL tasks in a phase before moving to the next.

> **RULE 6 — PLAN IS SOURCE OF TRUTH (EXECUTE MODE)**
>
> The planning documents define what to build, how to structure it, and which patterns to follow. Every implementation decision must trace back to the plan. Do not improvise architecture or invent new patterns.

> **RULE 7 — CONVENTIONS ARE NON-NEGOTIABLE**
>
> `discovery/CONVENTIONS.md` defines the project's DNA. Every planning proposal and every line of implementation code MUST follow these conventions.

> **RULE 8 — ASSUMPTION TRANSPARENCY**
>
> When information is missing: PLAN mode documents assumptions in ANALYSIS.md. EXECUTE mode documents decisions in the Decision Log. Never silently assume.

---

## Mode Detection

### Top-Level: PLAN vs EXECUTE

| Mode | Signals | Example Inputs |
|------|---------|----------------|
| **PLAN** | Planning, analyzing, designing, "plan", "create a plan", new work without existing plan | "Plan the auth system", "I need an app to track expenses" |
| **EXECUTE** | Implementing, executing, "start working", "implement the plan", existing plan output present | "Execute the project plan", "Start implementing Sprint 1" |

**Disambiguation**: If the user's intent is unclear, ask:

> "Do you want me to **plan** this work (produce documentation) or **execute** an existing plan (implement code)?"

### PLAN Sub-Modes

When in PLAN mode, detect the planning sub-mode. See [assets/modes/PLAN.md](assets/modes/PLAN.md) for the full detection table and capabilities matrix.

| Sub-Mode | Signals |
|----------|---------|
| **NEW_PROJECT** | Product idea, "build from scratch", no existing codebase |
| **NEW_FEATURE** | Adding to existing project, "add", "implement" |
| **REFACTOR** | Restructure, reorganize, migrate patterns |
| **BUG_FIX** | Fix, broken, regression, error |
| **TECH_DEBT** | Cleanup, dead code, deprecated, missing tests |
| **ARCHITECTURE** | System design, scaling, infrastructure change |

---

## Asset Loading (Mode-Gated)

After detecting the mode, read ONLY the assets listed for that mode. Do NOT read assets for other modes — they waste context tokens.

| Mode | Read These Assets | Do NOT Read |
|------|-------------------|-------------|
| **PLAN** | `output-resolve.md`, `PLAN.md`, detected sub-mode `.md` | EXECUTE.md, config-resolver.md, decision-log.md, code-quality-standards.md, unrelated sub-mode files, examples/, validators/ |
| **EXECUTE** | `output-resolve.md`, `EXECUTE.md`, `decision-log.md`, `code-quality-standards.md` | PLAN.md, config-resolver.md, all sub-mode files, templates/, examples/, validators/ |

**PLAN sub-mode loading**: After detecting the sub-mode from user input, read ONLY that file (e.g., `NEW_FEATURE.md`). Do NOT read the other 5 sub-mode files. Templates are loaded on-demand as each workflow step references them.

**On-demand helpers** (both modes): `troubleshooting.md` — read only when issues arise. `config-resolver.md` — loaded only when `output-resolve.md` can't find existing config (first-time setup).

---

## Quick Start

### PLAN Mode

Use when planning any software work:

> Plan the authentication system for the app.

**Assets to read now:** [output-resolve.md](assets/helpers/output-resolve.md) + [PLAN.md](assets/modes/PLAN.md) — then read only the detected sub-mode file from assets/modes/.

### EXECUTE Mode

Use when implementing an existing plan:

> Execute the project plan.

**Assets to read now:** [output-resolve.md](assets/helpers/output-resolve.md) + [EXECUTE.md](assets/modes/EXECUTE.md) + [decision-log.md](assets/helpers/decision-log.md) + [code-quality-standards.md](assets/helpers/code-quality-standards.md)

---

## Capabilities Matrix

| Capability | PLAN Mode | EXECUTE Mode |
|------------|:---------:|:------------:|
| Create planning documents | Yes | No |
| Write/modify code | No | Yes |
| Create CONVENTIONS.md | Yes | No (read-only) |
| Modify sprint files (status, checkboxes) | Yes (creation) | Yes (updates) |
| Modify PROGRESS.md | Yes (creation) | Yes (updates) |
| Run verification commands | No | Yes |
| Generate retrospectives | No | Yes (optional) |
| Make architectural decisions | Yes (in docs) | No (follow plan) |
| Log implementation decisions | No | Yes (Decision Log) |

---

## Mode Workflows (Quick Reference)

### PLAN Mode Workflow

1. **Configuration Resolution** → resolve `{output_dir}`
2. **Codebase Discovery** → `discovery/CONVENTIONS.md` (except NEW_PROJECT)
3. **Analysis** → `analysis/ANALYSIS.md`
4. **Planning** → `planning/PLANNING.md`
5. **Execution Plan** → `execution/EXECUTION.md`
6. **Sprint Plans** (optional) → `sprints/PROGRESS.md` + `sprints/SPRINT-*.md`
7. **Handoff** → summarize and stop

**Full details:** See [assets/modes/PLAN.md](assets/modes/PLAN.md)

### EXECUTE Mode Workflow

0. **Locate and Validate** → verify plan directory exists and is complete
1. **Internalize Plan** → read all planning documents
2. **Execute Active Sprint** → phase-by-phase, task-by-task
3. **Complete Sprint** → update tracking, verify graduation gates
3.5. **Retrospective** (optional) → capture learnings
4. **Project Completion** → final metrics and report

**Full details:** See [assets/modes/EXECUTE.md](assets/modes/EXECUTE.md)

---

## Configuration Resolution

`{output_dir}` is the directory where universal-planner stores all planning documents. Resolve it once at the start:

1. **User message context** — If the user's message contains file paths, extract `{output_dir}` from those paths
2. **Auto-discover** — Scan for `.agents/universal-planner/` in `{cwd}`
3. **Ask the user** — If nothing found, ask where to save documents. Default suggestion: `.agents/universal-planner/{project-name}/`

No AGENTS.md. No branded blocks. The output directory is resolved at runtime.

See [assets/helpers/output-resolve.md](assets/helpers/output-resolve.md) for the lightweight resolver. For full error handling, see [assets/helpers/config-resolver.md](assets/helpers/config-resolver.md).

---

## Obsidian Output Standard

All documents generated or modified by this skill MUST follow these rules:

1. **Frontmatter**: Every `.md` file includes the universal frontmatter schema (title, date, updated, project, type, status, version, tags, changelog, related)
2. **Types**: Use `conventions`, `analysis`, `plan`, `execution-plan`, `sprint-plan`, `progress`, `requirements`, `architecture`, `retrospective` as appropriate
3. **Wiki-links**: All inter-document references use `[[filename]]` syntax — never `[text](relative-path.md)`
4. **References**: Every document ends with `## References` listing Parent, Siblings, Children, and Input Documents
5. **Living documents**: PROGRESS.md and sprint plans are living documents — bump `version`, update `changelog`, transition `status` on modification
6. **IDs**: Use FR-/NFR- for requirements, ADR- for architecture decisions, T- for tasks, DEC- for decisions, K-/P-/L-/A- for retrospective items
7. **Bidirectional**: If document A references B, document B must reference A in `related` and `## Referencias`
8. **Metrics**: Use `| Metric | Before | After | Delta | Status |` format for all quantitative data
9. **Gates**: Sprint plans include `## Graduation Gate` with verifiable checkbox criteria
10. **Carried Forward**: Sprint 2+ includes `## Carried Forward from [[previous-sprint]]` for incomplete items
11. **Sequential numbering**: Use `01-`, `02-` prefixes where reading order matters
12. **Status transitions**: Move status through `draft → active → completed` (or `draft → active → failed`) as work progresses

**Frontmatter template for sprint plans:**
```yaml
---
title: "Sprint {N}: {Name}"
date: "YYYY-MM-DD"
updated: "YYYY-MM-DD"
project: "{project-name}"
type: "sprint-plan"
status: "draft"
version: "1.0"
sprint: {N}
phase: "{active-phases}"
progress: 0
previous_doc: "[[SPRINT-{N-1}-name]]"
next_doc: "[[SPRINT-{N+1}-name]]"
parent_doc: "[[PROGRESS]]"
tags: ["{project-name}", "sprint-{N}", "sprint-plan"]
changelog:
  - version: "1.0"
    date: "YYYY-MM-DD"
    changes: ["Initial sprint plan"]
related:
  - "[[PROGRESS]]"
  - "[[EXECUTION]]"
  - "[[SPRINT-{N-1}-name]]"
  - "[[SPRINT-{N+1}-name]]"
---
```

**Mode-specific frontmatter extensions:**
- **BUG_FIX**: Add `severity: "critical|high|medium|low"` to ANALYSIS.md frontmatter
- **REFACTOR**: Add `scope_modules: ["module-a", "module-b"]` to ANALYSIS.md frontmatter

---

## Sprint Structure Standards

All sprint documents follow the Obsidian-native markdown standard with structured frontmatter, progress tracking, and cross-references.

**Complete sprint formats:** See [assets/templates/SPRINT.md](assets/templates/SPRINT.md)

**Task granularity:**
- **L1 - Sprint**: Major milestone (3-7 days)
- **L2 - Phase**: Logical grouping (0.5-2 days)
- **L3 - Task**: Discrete work item (1-4 hours) with file paths, before/after code, verification
- **L4 - Subtask**: Single action (5-30 min)

Every task must include: checkbox, file paths (when applicable), before/after code snippets, verification command, and reference to existing patterns/components.

---

## Integration with Other Skills

| Skill | Integration |
|-------|------------|
| `code-analyzer` | Use before PLAN mode (REFACTOR or TECH_DEBT sub-mode) to get a detailed technical report as input |
| `skill-creator` | Reference when building skills that complement planning outputs |
| `obsidian` | Sync planning output to vault or read notes for context. Invoke via `Skill("obsidian")` or say "sync to obsidian". Subagent fallback: read obsidian SKILL.md directly. |

---

## Troubleshooting

Common issues and resolutions for both PLAN and EXECUTE modes.

**Full troubleshooting guide:** See [assets/helpers/troubleshooting.md](assets/helpers/troubleshooting.md)

---

## Limitations

1. **Mode boundary**: PLAN mode cannot execute; EXECUTE mode cannot plan
2. **Requires input**: PLAN mode needs a description of the work; EXECUTE mode needs plan output
3. **Assumptions must be validated**: Inferred decisions need team review before implementation
4. **Technology recommendations are suggestions**: The development team makes final choices
5. **Manual tracking**: Sprint progress must be updated manually during execution
6. **No automated validation**: Cannot verify plans match codebase reality — relies on thorough discovery
7. **Sequential execution**: EXECUTE mode processes one sprint at a time, in order
8. **External blockers**: Cannot resolve dependencies on external teams or services — logs them and moves on
9. **Context window**: For projects with more than 4 sprints, consider executing sprints in separate sessions to avoid context exhaustion. EXECUTE mode can resume from the last incomplete sprint.
