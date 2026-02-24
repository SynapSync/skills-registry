---
name: project-brain
description: >
  Session memory for AI agents — load context at the start, save sessions at the end,
  evolve knowledge across sessions. Like a professional's notebook: open before work,
  write a summary when done, persist between sessions.
  Trigger: When starting a session and need to recover context, or ending a session and
  want to save what happened.
license: Apache-2.0
metadata:
  author: synapsync
  version: "2.7"
  scope: [root]
  auto_invoke:
    # English triggers — LOAD
    - "load project brain"
    - "read project brain"
    - "restore context from"
    - "recover session context"
    - "load brain document"
    - "brief me on the project"
    - "what's the project state"
    - "load context from"
    - "pick up where we left off"
    # English triggers — SAVE
    - "save project brain"
    - "save session"
    - "update the brain"
    - "save what we did"
    - "write session summary"
    - "persist this session"
    # Spanish triggers — LOAD
    - "carga el brain del proyecto"
    - "lee el brain"
    - "recupera el contexto"
    - "ponme al dia del proyecto"
    - "carga el contexto de sesion"
    - "retoma la sesion desde"
    - "recupera la sesion anterior"
    # Spanish triggers — SAVE
    - "guarda el brain del proyecto"
    - "guarda la sesion"
    - "actualiza el brain"
    - "guarda lo que hicimos"
    - "escribe el resumen de sesion"
    - "persiste esta sesion"
allowed-tools: Read, Edit, Write, Glob, Grep, ToolSearch, AskUserQuestion
---

# Project Brain

## Assets

This skill uses a modular assets architecture. Detailed workflows, helpers, and templates are in the [assets/](assets/) directory:

- **[assets/modes/](assets/modes/)** — LOAD and SAVE mode workflows
- **[assets/helpers/](assets/helpers/)** — Brain directory config resolution, incremental merge algorithm
- **[assets/templates/](assets/templates/)** — Standard brain document format

See [assets/README.md](assets/README.md) for full directory documentation.

---

## Purpose

Project Brain is **session memory for AI agents**. It bridges the gap between sessions so any agent can pick up exactly where the last one left off — and save what it learned before shutting down.

**Design principle**: Agents behave like professionals with a notebook. Open the notebook before work (LOAD), write a summary when done (SAVE), and the notebook persists between sessions. Memory is a natural part of the workflow, not an extra step.

**What it does:**
- **LOAD**: Reads a brain document, parses structured context, delivers a concise briefing
- **SAVE**: Captures session context, creates or incrementally updates a brain document

**What a brain document is:**
A markdown file that captures project state, session history, architecture decisions, and next steps. The standard format (v2.0) has defined sections, but the skill also reads v1.0 and free-form documents.

---

## Critical Rules

> **RULE 1 — NO FABRICATION**
>
> Only report information that exists in the brain document (LOAD) or was explicitly discussed in the session (SAVE). Never infer, guess, or complete missing data.

> **RULE 2 — USER CONFIRMS BEFORE WRITE**
>
> In SAVE mode, always present the gathered session data to the user for review before writing. Never write without confirmation.

> **RULE 3 — INCREMENTAL NOT DESTRUCTIVE**
>
> SAVE UPDATE merges new data into existing sections. It never overwrites the full document. Accumulated Context is append-only. Session Log is prepend-only.

> **RULE 4 — FORMAT BACKWARD-COMPATIBLE**
>
> LOAD parses three formats: v2.0 standard, v1.0 sections, and free-form markdown. SAVE UPDATE detects the format and offers migration if the document isn't v2.0.

> **RULE 5 — PATH RESOLUTION ONCE**
>
> Resolve `{brain_dir}` once per session via AGENTS.md config (see Configuration Resolution). The resolved path is used for all operations. Don't re-ask unless the user wants to change the path.

> **RULE 6 — LANGUAGE SEPARATION**
>
> **Conversation** (briefings, confirmations, questions) → respond in the same language the user used.
> **Brain document content** (what gets written to file) → always in English, regardless of conversation language. Technical artifacts must be consistent across sessions.

> **RULE 7 — NO SILENT DEFAULTS**
>
> When `{brain_dir}` is not configured in AGENTS.md, you MUST ask the user via `AskUserQuestion` before proceeding. Never silently choose the default staging path. The user must actively confirm their preferred directory.

---

## Configuration Resolution

Before starting any mode workflow, resolve `{brain_dir}` — the directory where brain documents are stored.

1. **Read** `{cwd}/AGENTS.md` → scan for `<!-- synapsync-skills:start -->` block → find `## Configuration` table → parse `brain_dir` row
2. If `brain_dir` found → use it, done
3. If not found → **ask the user**:
   - Option A: **Use default** (`.agents/staging/project-brain/`)
   - Or select "Other" to write a custom path
4. **Persist** the chosen value to AGENTS.md Configuration table

Full resolution algorithm: see [assets/helpers/brain-config.md](assets/helpers/brain-config.md)

---

## Mode Detection

| Mode | EN Signals | ES Signals | What It Does |
|------|-----------|-----------|-------------|
| **LOAD** | "load brain", "restore context", "brief me", "pick up where we left off" | "carga el brain", "recupera el contexto", "ponme al dia" | Auto-discovers brain, reads it, delivers context briefing |
| **SAVE** | "save brain", "save session", "update brain", "persist session" | "guarda el brain", "guarda la sesion", "actualiza el brain" | Gathers session data, creates or updates brain document |

**Disambiguation**: If the user's intent is unclear, ask:

> "Do you want me to **load** a brain document (restore context) or **save** the current session (capture what we did)?"

---

## Asset Loading (Mode-Gated)

After detecting the mode, read ONLY the assets listed for that mode. Do NOT read assets for other modes — they waste context tokens.

| Mode | Read These Assets | Do NOT Read |
|------|-------------------|-------------|
| **LOAD** | `assets/modes/LOAD.md`, `assets/helpers/brain-resolve.md` | SAVE.md, brain-config.md, incremental-merge.md, BRAIN-DOCUMENT.md |
| **SAVE INIT** | `assets/modes/SAVE.md`, `assets/helpers/brain-config.md`, `assets/templates/BRAIN-DOCUMENT.md` | LOAD.md, brain-resolve.md, incremental-merge.md |
| **SAVE UPDATE** | `assets/modes/SAVE.md`, `assets/helpers/brain-config.md`, `assets/helpers/incremental-merge.md` | LOAD.md, brain-resolve.md, BRAIN-DOCUMENT.md |

Read the mode asset first, then follow its internal references to helpers/templates as needed.

---

## Quick Start

### LOAD Mode

Use at the start of a session to restore context:

> Load the project brain.

This will: resolve `{brain_dir}` from AGENTS.md, scan for existing documents, read the brain, parse sections, and deliver a context briefing.

**Assets to read now:** [assets/modes/LOAD.md](assets/modes/LOAD.md) + [brain-resolve.md](assets/helpers/brain-resolve.md)

### SAVE Mode

Use at the end of a session to persist what happened:

> Save the session to the brain.

This will: detect if a brain exists (UPDATE) or not (INIT), gather session data, confirm with you, and write or merge the brain document.

**Assets to read now:** [assets/modes/SAVE.md](assets/modes/SAVE.md) + [brain-config.md](assets/helpers/brain-config.md) (SAVE.md references additional helpers per sub-mode)

---

## Capabilities Matrix

| Capability | LOAD | SAVE (INIT) | SAVE (UPDATE) |
|-----------|:----:|:-----------:|:-------------:|
| Resolve `{brain_dir}` from AGENTS.md | Yes | Yes | Yes |
| Auto-discover brain documents | Yes | No | Yes |
| Read brain from Obsidian MCP | Yes | No | No |
| Read brain from filesystem | Yes | No | No |
| Parse v2.0, v1.0, free-form | Yes | No | Yes |
| Deliver context briefing | Yes | No | No |
| Create new brain document | No | Yes | No |
| Incremental merge | No | No | Yes |
| Gather session context | No | Yes | Yes |
| Offer format migration | No | No | Yes |

---

## Integration with Other Skills

| Skill | Integration |
|-------|------------|
| `sprint-forge` | LOAD reads sprint-forge re-entry prompts. SAVE captures sprint progress as session entries. |
| `obsidian` | LOAD can read brain documents from vault via obsidian READ mode. Invoke via `Skill("obsidian")` or say "read from obsidian". Subagent fallback: read obsidian SKILL.md directly. Never call `mcp__obsidian__*` without the skill. |
| `universal-planner` | LOAD reads planning documents. SAVE captures planning decisions. |

**Composition patterns:**
```
session start  → project-brain LOAD  → agent is briefed → work happens
session end    → project-brain SAVE  → session persisted → next agent can LOAD
```

```
sprint-forge INIT  → generates findings + roadmap
project-brain SAVE → captures sprint-forge session → brain document created
new session        → project-brain LOAD → full context restored
```

---

## Limitations

1. **Markdown only**: Handles `.md` files; does not parse `.yaml`, `.json`, or other formats
2. **Single file**: One brain document per project — not a recursive folder reader
3. **No synthesis**: If the brain document is outdated, the briefing reflects that
4. **MCP for Obsidian only in LOAD**: SAVE always writes to filesystem (Obsidian syncs via the `obsidian` skill if needed)
5. **Session Log size**: Compacted at 15+ entries — older sessions are archived to `{brain_dir}/archive/` and replaced with a summary paragraph
6. **No auto-save**: SAVE must be explicitly invoked — the agent doesn't auto-save on exit
