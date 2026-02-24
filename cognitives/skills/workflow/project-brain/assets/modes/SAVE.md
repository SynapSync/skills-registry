# SAVE Mode

Saves session context to a brain document. Two sub-modes: **INIT** (create new) and **UPDATE** (merge into existing).

---

## Step 0 — Resolve Brain Directory

Resolve `{brain_dir}` using the [brain-config helper](../helpers/brain-config.md). This determines where brain documents are stored before any file operations.

**IMPORTANT**: If no `brain_dir` is configured in AGENTS.md, the helper requires you to ask the user where to store brain documents via `AskUserQuestion`. Never assume the default path.

After this step, `{brain_dir}` is set (e.g., `.agents/project-brain` or a custom path from AGENTS.md).

---

## Step 1 — Detect Sub-Mode

1. Check `{cwd}/{brain_dir}/` for existing `.md` files
2. If **at least one file exists** → **UPDATE** (ask which file if multiple)
3. If **no files exist** → **INIT**

The user can override: "save as new brain" forces INIT even if one exists.

---

## Step 2 — Resolve Output Path (INIT only)

Ask before the first write:

> "Where should I save the brain document for **{project_name}**?
>
> 1. **Local** (default) — `{brain_dir}/{project-name}.md`
> 2. **Custom path** — provide a root and I'll save to `{root}/{project-name}.md`"

Path resolution:
- **Option 1**: `{cwd}/{brain_dir}/{project-name}.md` — create `{brain_dir}/` if needed
- **Option 2**: `{user-root}/{project-name}.md` — save directly, no `project-brain/` namespace. After writing, persist the custom `{brain_dir}` to AGENTS.md if not already done (see [brain-config helper](../helpers/brain-config.md)).

`{project-name}` is inferred from the current directory name or git repository name (kebab-case).

For UPDATE, the path is already known from Step 1.

---

## Step 3 — Gather Session Context

**Session ID**: Generate a Session ID at the start: `SID: {YYYYMMDD-HHMM}` using the current date and time.

Synthesize from the current conversation:

1. **Summary**: 2-3 sentences of what was accomplished this session
2. **Decisions**: Explicit decisions made (architecture choices, library selections, approach changes)
3. **Discoveries**: Non-obvious findings (bugs found, performance insights, undocumented behaviors)
4. **Files Changed**: Run `git diff --name-only` if git repo; otherwise ask the user
5. **Retrospective**: Process reflection — friction points, approach pivots, time sinks, and what worked well. Focus on things NOT already captured in Decisions or Discoveries (e.g., "started with approach X but switched to Y because Z", "biggest time sink was debugging the API integration because docs were outdated"). If the session was straightforward with no notable friction or pivots, write `Straightforward session.` — do NOT fabricate reflections.
6. **Next Steps**: Only include items that were explicitly discussed or requested during the session. If no next steps were discussed, write `No explicit next steps discussed.` — do NOT infer or fabricate items to fill this section.

For **INIT** only, also gather:
7. **Project Identity**: Name, purpose, stack, repository, key paths — infer from cwd or ask
8. **Key Files**: Critical project files and why they matter

### Confirmation Gate

**Present all gathered data to the user for review before writing.** Format it clearly:

```
Here's what I'll save to the brain document:

**Summary**: {summary}
**Decisions**: {list}
**Discoveries**: {list}
**Files Changed**: {list}
**Retrospective**: {reflection or "Straightforward session."}
**Next Steps**: {list or "No explicit next steps discussed."}

{For INIT: also show Project Identity and Key Files}

Confirm? I can adjust anything before saving.
```

Do NOT write until the user confirms.

---

## Step 4 — Write (INIT) or Merge (UPDATE)

### INIT Sub-Mode

1. Fill the BRAIN-DOCUMENT template (see [templates/BRAIN-DOCUMENT.md](../templates/BRAIN-DOCUMENT.md)):
   - Set all `{placeholders}` with gathered data
   - Session Log starts with `### Session 1 — {today's date}`
   - Accumulated Context starts empty (just headers) unless the user provided architecture decisions
   - Set metadata: `Last updated: {today}`, `Sessions: 1`
2. Write the file to the resolved path
3. After writing, persist `{brain_dir}` to AGENTS.md if not already done (see [brain-config helper](../helpers/brain-config.md))

### UPDATE Sub-Mode

Use the incremental merge algorithm (see [helpers/incremental-merge.md](../helpers/incremental-merge.md)):

**Pre-write backup**: Before reading the document for merge, copy the current brain document:
> `{path}` → `{path}.bak`
>
> Use the Write tool to copy the contents. If the merge produces incorrect results, the user can restore from the `.bak` file.

1. Read the existing brain document
2. Detect format:
   - **v2.0** → proceed with merge
   - **v1.0 or free-form** → offer migration first (see incremental-merge helper)
3. Apply per-section merge:

| Section | Strategy |
|---------|----------|
| Metadata (date, session count) | Update |
| Project Identity | Update only changed fields |
| Active State | Full replace |
| Session Log | Prepend new entry (check SID for duplicates first — see below) |
| Accumulated Context > Architecture Decisions | Append rows |
| Accumulated Context > Patterns & Conventions | Append (skip duplicates) |
| Accumulated Context > Constraints & Gotchas | Append (skip duplicates) |
| Next Steps | Full replace |
| Key Files | **Merge + staleness check**: add new, update changed, keep rest. For each existing entry, verify the file still exists (using Glob). If a file was deleted, remove the entry or mark it as `~~{path}~~ (removed)` and confirm with user. |

**Session ID duplicate check**: Before prepending the new session entry, check if an entry with the same SID already exists in Session Log. If found:
> "Session already saved (SID: {SID}). Update the existing entry or create a new one?"

**Contradiction check**: When appending to Accumulated Context, check if the new item contradicts an existing one (e.g., "Use library X" vs "Stopped using library X"). If a contradiction is detected:
> "Pattern '{new}' seems to contradict existing '{old}'. Retire the old one?"
>
> If yes → move the old item to a `### Retired` sub-section at the bottom of Accumulated Context.
> If no → keep both items.

4. Write the merged document

---

## Step 5 — Confirm

Show a summary of what was saved:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
  PROJECT BRAIN SAVED
  Path: {path}
  Session: {N} | Date: {today}
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

{For INIT:}
Created new brain document with Session 1.

{For UPDATE:}
Added Session {N}.
Updated: Active State, Next Steps{, N new architecture decisions}{, N new patterns}.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

No further steps. The file is already where the user chose to save it.
