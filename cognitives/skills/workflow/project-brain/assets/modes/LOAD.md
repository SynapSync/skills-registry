# LOAD Mode

Loads a brain document and delivers a structured context briefing. Supports Obsidian vault paths, filesystem paths, and three document formats (v2.0, v1.0, free-form).

---

## Step 0 — Resolve Brain Directory

Resolve `{brain_dir}` using the [brain-resolve helper](../helpers/brain-resolve.md).

After this step, `{brain_dir}` is set (e.g., `.agents/project-brain/my-app` or a custom path from the user).

---

## Step 1 — Discover Brain Document

Scan `{cwd}/{brain_dir}/` for `.md` files:

1. If **one file found** → offer it:
   > "Found brain: `{brain_dir}/{name}.md` — Load this?"
   - If yes → proceed to Step 3 with that path
   - If no → Step 2
2. If **multiple files found** → ask which one:
   > "Found {N} brain documents in `{brain_dir}/`:
   > 1. `{name1}.md`
   > 2. `{name2}.md`
   > Which one should I load?"
3. If **no files found** → Step 2

---

## Step 2 — Resolve Source Path

If no brain documents were found in `{brain_dir}`, or the user declined, ask for the source:

```
AskUserQuestion:
  question: "Where is the project brain document?"
  header: "Brain source"
  options:
    - label: "Obsidian vault"
      description: "Provide a vault-relative path (e.g. joicodev/project/brain.md)"
    - label: "Filesystem path"
      description: "Provide an absolute or relative path to the .md file"
    - label: "Create new"
      description: "No brain exists yet — switch to SAVE mode to create one"
```

Path classification:
- **Obsidian vault** → expand to absolute path using the configured vault root (from CLAUDE.md or ask the user) and use `Read`.
- **Filesystem path** → resolve relative paths from cwd, use `Read` directly
- **Create new** → switch to SAVE mode (INIT sub-mode) and stop LOAD

---

## Step 3 — Read + Parse

Read the document using the resolved access method.

If the file does not exist or returns an error:
- Report the exact error to the user
- Ask if they want to provide a different path
- Do NOT guess alternative paths

### Parsing Priority

Parse the document in order of format priority:

**Priority 0 — Format Marker:**
Check the first 5 lines for `<!-- brain-format: vX.X -->`. If found, use that version directly and skip the heading heuristic.

**Priority 1 — v2.0 Standard Sections:**
Look for these h2 headings:
- `## Project Identity`
- `## Active State`
- `## Session Log`
- `## Accumulated Context`
- `## Next Steps`
- `## Key Files`

If 3+ of these are found → treat as v2.0 format.

**Priority 2 — v1.0 Sections:**
Look for these h2 headings (EN and ES):
- `## Project Overview` / `## Resumen del Proyecto`
- `## Current State` / `## Estado Actual`
- `## Progress` / `## Completed` / `## Progreso` / `## Completado`
- `## Pending Tasks` / `## Next Steps` / `## Tareas Pendientes` / `## Próximos Pasos`
- `## Key Decisions` / `## Decisiones Clave`
- `## Blockers` / `## Bloqueadores`
- `## Technical Notes` / `## Notas Técnicas`
- `## Context` / `## Contexto`

If 2+ of these are found → treat as v1.0 format.

**Priority 3 — Free-Form:**
Any `.md` file that doesn't match the above. Read the full document and derive structured sections from its content.

---

## Step 4 — Briefing

Present a concise context briefing. Only show what's essential for starting work — don't dump the entire document.

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PROJECT BRAIN LOADED
  Source: {path}
  Sessions: {N} | Last: {date}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

## Project Identity
{name, purpose, stack — from table or inferred}

## Active State
{current phase, WIP, blockers, next priority}

## Last Session ({date})
{summary, decisions, discoveries from most recent session entry}

## Next Steps
{prioritized list}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  Context restored.
  Suggestion: {first Next Step}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

**What is NOT in the briefing** (but available on demand during the session):
- Full Session Log history
- Accumulated Context (Architecture Decisions, Patterns, Constraints)
- Key Files table

For **v1.0 documents**, map sections to the briefing:
- `Project Overview` → Project Identity
- `Current State` → Active State
- Most recent `Progress` entry → Last Session
- `Pending Tasks` / `Next Steps` → Next Steps

For **free-form documents**, derive what you can and present it. Mark inferred sections explicitly: `(inferred from document)`.

After the briefing:

```
Context loaded from: {path}
I'm briefed on the project. What would you like to work on?
```

If the brain document includes explicit next steps, surface the first one as a suggestion.
