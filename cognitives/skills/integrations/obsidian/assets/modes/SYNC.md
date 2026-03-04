# SYNC Mode - Write to Obsidian Vault

## When to Use This Mode

- User explicitly asks to **sync**, **save**, **move**, or **store** files to Obsidian
- (ES) User says "guardar en obsidian", "sincronizar a obsidian", or similar in Spanish
- User asks to **create a report/plan AND save it to Obsidian** (this mode handles the Obsidian part after creation)
- User wants to **browse their vault** to choose where to place documents
- User asks to move an entire folder (e.g., `{output_dir}/planning/`) to their Obsidian vault
- After another skill finishes generating documents, user wants them in Obsidian

## When NOT to Use This Mode

- User wants to read or search notes already in Obsidian (use READ mode)
- User wants to edit an existing Obsidian note in-place (use MCP tools directly)
- Files are not markdown or text-based (this mode is for `.md` content)

---

## Critical Rules

**RULE 0 - USE THE EXACT PATH THE USER GIVES YOU**

When the user specifies a vault destination path, use it **exactly as given**. Do not modify it, do not substitute it, do not "improve" it. The `.agents/staging/` pattern found in skill examples is an `output_dir` convention for other skills — it is NOT a vault folder and must NEVER be used as a vault destination. If the user says "save to `work/my-project/plans/`", write to `work/my-project/plans/` — never to `agents/`, `agents/{project-name}/`, or any variation.

**RULE 1 - ALWAYS ASK WHERE TO SAVE**

Never assume the destination folder. Always:
1. List the vault's root directories
2. Let the user pick or specify the destination
3. If the user already specified a path, use it directly — confirm it exists or offer to create it
4. NEVER default to `agents/` or any variation as a vault folder

**RULE 2 - PRESERVE CONTENT INTEGRITY**

- Read the source file completely before writing to Obsidian
- Never modify the document body content (headings, paragraphs, tables, code blocks, `## Referencias` section)
- Only ADD or MERGE frontmatter metadata — never edit anything below the closing `---`
- If the file already has frontmatter, merge Obsidian-specific fields without overwriting existing ones
- Cross-reference fixes (via cross-ref-validator) update the `related` frontmatter array only

**RULE 3 - GENERATE MEANINGFUL FRONTMATTER**

Every note written to Obsidian MUST include frontmatter following the universal schema:

```yaml
---
title: "Document Title"
date: "YYYY-MM-DD"
updated: "YYYY-MM-DD"
project: "{project-name}"
type: "{document-type}"
status: "active"
version: "1.0"
tags: [relevant, tags, here]
changelog:
  - version: "1.0"
    date: "YYYY-MM-DD"
    changes: ["Synced to Obsidian"]
related:
  - "[[related-document]]"
---
```

Infer these values from:
- `title`: First H1 heading in the document, or filename
- `date`: File modification date, or today's date
- `updated`: Today's date (sync date)
- `project`: Current working directory name or git repo name
- `type`: Infer from content using the 14-type taxonomy (see expanded type inference table below)
- `status`: `"active"` for new documents, preserve existing status
- `version`: `"1.0"` for new documents, preserve existing version
- `tags`: Infer from project name, document type, and key topics
- `changelog`: Add sync entry
- `related`: Extract from `[[wiki-links]]` found in document content, or from `## Referencias` section

**RULE 4 - REPORT RESULTS**

After syncing, always report:
- How many files were synced
- The exact Obsidian paths where they were saved
- Any files that were skipped and why

---

## Obsidian Output Standard

When syncing documents to Obsidian, follow the [Obsidian markdown standard](../standards/obsidian-md-standard.md) Sync rules:

1. **Frontmatter enrichment**: If a document has minimal frontmatter (only title/date/project/type/tags), enrich it with the full universal schema (add updated, status, version, changelog, related)
2. **Frontmatter preservation**: If a document already has rich frontmatter, preserve all existing fields — only add missing ones
3. **Frontmatter generation**: If a document has no frontmatter, generate the complete universal schema (see [../helpers/frontmatter-generator.md](../helpers/frontmatter-generator.md))
4. **Wiki-link preservation**: Never convert `[[wiki-links]]` to `[markdown](links)` — preserve them exactly
5. **Type inference**: Map document content to the 14-type taxonomy defined in the [Obsidian markdown standard](../standards/obsidian-md-standard.md)
6. **Cross-reference validation**: After syncing a batch of files, verify that `related` fields and `## Referencias` sections are bidirectional (see [../helpers/cross-ref-validator.md](../helpers/cross-ref-validator.md))
7. **Status**: New documents get `status: "active"`, documents with existing status are preserved

---

## Workflow

### Step 0: Detect Access Mode

Before any vault operation, determine the access strategy:

```
# Attempt MCP connection
ToolSearch query: "+obsidian write"
ToolSearch query: "+obsidian list"
```

| Result | Mode | Behavior |
|--------|------|----------|
| Tools loaded successfully | **MCP Mode** | Use `mcp__obsidian__*` tools for all operations |
| ToolSearch returns no obsidian tools | **Fallback Mode** | Ask user for vault path, use Write/Read/Edit/Glob/Bash |
| Tools load but fail on call | **Fallback Mode** | Warn user MCP may be disconnected, switch to filesystem |

**If entering Fallback Mode**, inform the user:

```
The Obsidian MCP server doesn't appear to be connected. I can still write to your vault
directly on the filesystem. What is the path to your Obsidian vault?
(e.g., ~/Documents/MyVault or ~/obsidian-vault)
```

Or use `AskUserQuestion`:
```
AskUserQuestion:
  question: "What is the path to your Obsidian vault?"
  header: "Vault path"
  options:
    - label: "~/obsidian-vault"
      description: "Common default (macOS/Linux)"
    - label: "~/Documents/Obsidian"
      description: "Documents folder (macOS/Linux)"
    - label: "C:\\Users\\{username}\\Documents\\Obsidian"
      description: "Documents folder (Windows)"
```

### Step 1: Identify What to Sync

Use `Glob` to discover source files based on user request (specific file, directory, pattern, or recent output).

### Step 2: Load Obsidian MCP Tools (MCP Mode Only)

**MCP Mode:** Load required tools: `ToolSearch query: "+obsidian list"` and `ToolSearch query: "+obsidian write"`

**Fallback Mode:** Skip this step — vault path was already obtained in Step 0.

### Step 3: Browse Vault and Present Options

**MCP Mode:** List vault structure with `mcp__obsidian__list_directory`, build folder tree, present options via `AskUserQuestion`. If user specified destination, use it directly. Always include "Create new folder" option.

**Fallback Mode:**
```
Glob(pattern: "{vault_path}/*")
# Or for deeper browsing:
Bash(command: "find {vault_path} -maxdepth 2 -type d | head -30")
```
Build folder tree from results, present options via `AskUserQuestion`.

### Step 4: Read Source Files

Read all files in parallel using `Read` tool. Check for existing frontmatter, extract metadata.

### Step 5: Write to Obsidian

For each file, call `mcp__obsidian__write_note` with:

> **Important**: `content` is the raw markdown body **without** the `---` frontmatter block. `frontmatter` is a **separate JSON object** — the MCP server serializes it as YAML and prepends it to the content automatically. If the `path` includes directories that don't exist, they are created.

**Write modes:**

| Mode | Behavior | When to Use |
|------|----------|-------------|
| `"overwrite"` (default) | Replace entire note | New syncs, full document updates |
| `"append"` | Add content to end of note | Adding entries to a log, appending sections |
| `"prepend"` | Add content to beginning of note | Adding notices or headers to existing notes |

> Always use `"overwrite"` for standard sync operations. Use `"append"` only when explicitly adding to an existing note.

```
mcp__obsidian__write_note(
  path: "{vault-destination}/{filename}.md",
  content: "{markdown-body-without-frontmatter-block}",
  mode: "overwrite",
  frontmatter: {
    "title": "Document Title",
    "date": "2026-02-10",
    "updated": "2026-02-12",
    "project": "agent-sync-sdk",
    "type": "analysis",
    "status": "active",
    "version": "1.0",
    "source": "{output_dir}/planning/2026-02-10/00-strategic-analysis.md",
    "tags": ["agent-sync-sdk", "analysis", "strategy"],
    "changelog": [{"version": "1.0", "date": "2026-02-12", "changes": ["Synced to Obsidian"]}],
    "related": ["[[01-technical-debt]]", "[[02-growth-vision]]"]
  }
)
```

**Fallback Mode:**

When writing without MCP, you must manually serialize frontmatter to YAML and prepend it to the content:

1. Generate the frontmatter object using the frontmatter-generator helper
2. Serialize it to YAML format
3. Construct the full file content: `---\n{yaml}\n---\n\n{body}`
4. Ensure the vault directory exists: `Bash(command: "mkdir -p {vault_path}/{destination}")`
5. Write the file: `Write(file_path: "{vault_path}/{destination}/{filename}.md", content: "{full_content}")`

**YAML Serialization Pattern:**

Given a frontmatter object:
```json
{
  "title": "Document Title",
  "date": "2026-02-10",
  "updated": "2026-02-14",
  "project": "my-project",
  "type": "analysis",
  "status": "active",
  "version": "1.0",
  "tags": ["tag1", "tag2"],
  "changelog": [{"version": "1.0", "date": "2026-02-14", "changes": ["Synced to Obsidian"]}],
  "related": ["[[doc-a]]", "[[doc-b]]"]
}
```

Serialize as:
```yaml
---
title: "Document Title"
date: "2026-02-10"
updated: "2026-02-14"
project: "my-project"
type: "analysis"
status: "active"
version: "1.0"
tags:
  - tag1
  - tag2
changelog:
  - version: "1.0"
    date: "2026-02-14"
    changes:
      - "Synced to Obsidian"
related:
  - "[[doc-a]]"
  - "[[doc-b]]"
---
```

Then prepend to body content and use `Write` to save to `{vault_path}/{destination}/{filename}.md`.

**Frontmatter generation:**

See [../helpers/frontmatter-generator.md](../helpers/frontmatter-generator.md) for the complete frontmatter generation workflow.

**Type inference:**

Refer to the [Obsidian markdown standard](../standards/obsidian-md-standard.md) for the 14-type document taxonomy used in type inference.

**Wiki-link handling:**
- Preserve all existing `[[wiki-links]]` in document content exactly as-is
- Never convert `[[wiki-links]]` to `[markdown](relative-links)`
- If a document contains `## Referencias` section, preserve it and add to `related` frontmatter field

### Step 5a: Small Edits with patch_note (Optional)

When a sync involves only a small change to an existing note (e.g., updating a status, fixing a typo), use `patch_note` instead of rewriting the entire note:

```
mcp__obsidian__patch_note(
  path: "work/agent-sync-sdk/plans/PROGRESS.md",
  oldString: "status: IN_PROGRESS",
  newString: "status: COMPLETED",
  replaceAll: false
)
```

**When to use `patch_note` vs `write_note`:**
- **patch_note**: Changing 1-3 specific strings in an existing note. Faster, preserves the rest exactly.
- **write_note**: New notes, full re-syncs, or changes affecting >30% of the document.

> With `replaceAll: false` (default), `patch_note` fails if `oldString` matches more than once, preventing unintended replacements. Set `replaceAll: true` only when you want to replace every occurrence.

**Fallback Mode:**
```
Read(file_path: "{vault_path}/{file}")
# Find the exact string to replace
Edit(file_path: "{vault_path}/{file}", old_string: "status: IN_PROGRESS", new_string: "status: COMPLETED")
```

### Step 5.5: Cross-Reference Validation

After syncing a batch of files (2+ files), validate bidirectional references.

See [../helpers/cross-ref-validator.md](../helpers/cross-ref-validator.md) for the complete cross-reference validation workflow.

> **Optimization (v3.2)**: The cross-ref validator uses `mcp__obsidian__update_frontmatter` to fix missing reverse references instead of reading the full note and rewriting it. This is faster and safer since it only touches metadata.

**Example output:**
```
Cross-reference validation:
- SPRINT-1-foundation.md references [[PROGRESS]] ✓ (PROGRESS references back)
- ANALYSIS.md references [[CONVENTIONS]] ✓ (CONVENTIONS references back)
- PLANNING.md references [[ANALYSIS]] ⚠ Fixed: used update_frontmatter to add [[PLANNING]] to ANALYSIS related
```

### Step 6: Report Results

After all files are written, report a summary:

```
Synced to Obsidian:
- work/agent-sync-sdk/plans/00-strategic-analysis.md
- work/agent-sync-sdk/plans/01-technical-debt.md
- work/agent-sync-sdk/plans/02-growth-vision.md

3 files synced successfully to vault folder: work/agent-sync-sdk/plans/
```

---

## Batch Sync Pattern

When syncing an entire folder, process files efficiently to minimize latency and user interaction.

See [../helpers/batch-sync-pattern.md](../helpers/batch-sync-pattern.md) for the complete optimized batch sync workflow.

**Key principles:**
1. **Glob** all `.md` files once
2. **Read** all files in parallel
3. **Load** MCP tools once (MCP Mode) or use vault path (Fallback Mode)
4. **List** vault directory once
5. **Ask** user once for destination
6. **Write** all files sequentially
7. **Validate** cross-references after all writes
8. **Report** all results at the end

---

## Integration with Other Skills

This mode is the **output handler** for planning, analysis, and reporting skills. Composition pattern: producer skill generates documents locally → obsidian SYNC mode picks them up and syncs to vault. User says "create X and save to obsidian" — both steps happen automatically.

---

## Configuration Options

**Sync Scope**: Single file, directory, glob pattern, or output from another skill.
**Destination Logic**: Use user-specified path, suggest existing project folder, offer to create new folder, or list multiple projects for selection.

---

## Output Format Contract

Every SYNC operation MUST produce a response with these sections in this order:

```markdown
### Source Files
- {N} files detected in `{source_path}`
- [list each filename]

### Destination
- Vault folder: `{vault_destination}`
- [Created / Already existed]

### Write Results
| File | Status | Vault Path |
|------|--------|------------|
| {filename} | Synced | {vault_path} |
| {filename} | Skipped (already exists) | — |
| {filename} | Failed: {reason} | — |

### Cross-Reference Validation
- {N} bidirectional references verified
- {N} frontmatter fixes applied
- [list each fix if any]

### Warnings
- [any overwrites, missing sources, MCP issues — or "None"]
- Operating in filesystem fallback mode (no MCP) — if applicable
```

**Rules:**
- Always include all 5 sections, even if empty (use "None" or "N/A")
- Never omit Write Results — the user must know exactly what was written and where
- Failed writes must include the reason

---

## Best Practices

- **Before Sync**: Verify source files exist, access mode detected (MCP or fallback), producer skill completed
- **During Sync**: Never modify content (only frontmatter), process writes sequentially, warn before overwriting
- **After Sync**: Report all synced paths, any failures, and suggest visual verification in Obsidian

---

## Optional Workflow: Batch Move from Staging

When the user asks to "sync my output to vault" or "move staging to obsidian", this workflow detects local staging directories and syncs their contents to the vault.

### When to Use

- User says "sync my staging output to obsidian" or "move my reports to vault"
- After a producer skill (universal-planner, code-analyzer) completes and the user wants to persist output in the vault

### Workflow

1. **Detect staging directories**: `Glob(pattern: ".agents/staging/*/")` to find all skill staging dirs
2. **List available skills**: Present the user with a list of skills that have staging output
3. **User picks**: Which skill's output to sync (or "all")
4. **For each selected skill's staging dir**:
   a. Glob all `.md` files in the staging directory
   b. Follow the standard SYNC workflow (Steps 1-6) to write to vault
   c. After successful sync, ask user if they want to clean up the staging directory
5. **Report**: Standard SYNC output format with all synced files

### Example

```
User: "sync my planning output to obsidian"

Detected staging output:
  - .agents/staging/universal-planner/my-app/ (8 files)
  - .agents/staging/code-analyzer/my-app/ (2 files)

Which output would you like to sync?
1. universal-planner (8 files)
2. code-analyzer (2 files)
3. All (10 files)

User selects: 3 (All)

→ Follows standard SYNC workflow for all 10 files
→ Reports results
→ "Would you like to clean up the staging directories?"
```

---

## Optional Workflow: Archive and Delete

When the user asks to clean up, archive, or remove vault notes.

> **DELETE is destructive and irreversible.** Always confirm with the user before deleting.

### Archive (Preferred)

1. **Move note to archive folder** using `move_note`:
   ```
   mcp__obsidian__move_note(
     oldPath: "work/agent-sync-sdk/plans/old-plan.md",
     newPath: "archive/agent-sync-sdk/old-plan.md",
     overwrite: false
   )
   ```
2. **Update frontmatter status** to `archived`:
   ```
   mcp__obsidian__update_frontmatter(
     path: "archive/agent-sync-sdk/old-plan.md",
     frontmatter: { "status": "archived", "updated": "2026-02-13" },
     merge: true
   )
   ```
3. **Report** the move with old and new paths.

### Delete (Destructive)

1. **Confirm with user** via `AskUserQuestion`:
   ```
   question: "Are you sure you want to permanently delete 'old-plan.md'? This cannot be undone."
   options:
     - "Yes, delete permanently"
     - "No, move to archive instead"
   ```
2. **If confirmed**, call `delete_note` with matching confirmation:
   ```
   mcp__obsidian__delete_note(
     path: "work/agent-sync-sdk/plans/old-plan.md",
     confirmPath: "work/agent-sync-sdk/plans/old-plan.md"
   )
   ```
3. **Report** the deletion and warn about broken references.

**Fallback Mode (Archive):**
```
Bash(command: "mkdir -p {vault_path}/archive/{project}")
Bash(command: "mv '{vault_path}/{old_path}' '{vault_path}/archive/{project}/{filename}'")
# Then update frontmatter manually:
Read(file_path: "{vault_path}/archive/{project}/{filename}")
# Parse frontmatter, change status to "archived", update "updated" field
Write(file_path: "{vault_path}/archive/{project}/{filename}", content: "{updated_content}")
```

**Fallback Mode (Delete):**
```
# Always confirm with user first via AskUserQuestion
Bash(command: "rm '{vault_path}/{file}'")
```

---

## Optional Workflow: Move and Reorganize

When the user asks to reorganize vault structure, move notes between folders, or rename notes.

### Single Note Move
```
mcp__obsidian__move_note(
  oldPath: "inbox/unsorted-note.md",
  newPath: "work/agent-sync-sdk/plans/sorted-note.md",
  overwrite: false
)
```

### Batch Reorganization

1. **List source directory**: `mcp__obsidian__list_directory(path: "inbox/")`
2. **Get metadata** for all notes: `mcp__obsidian__get_notes_info(paths: [...])`
3. **Ask user** for destination mapping
4. **Move notes sequentially**: `mcp__obsidian__move_note(...)` for each
5. **Update cross-references** if filenames changed:
   - Use `mcp__obsidian__search_notes(query: "[[old-name]]")` to find referencing notes
   - Use `mcp__obsidian__patch_note` to update wiki-links in other notes
6. **Report** all moves with old and new paths

> **Caution**: Moving a note breaks existing `[[wiki-links]]` if the filename changes. Obsidian handles this automatically if "Automatically update internal links" is enabled. If not, use `patch_note` to update references.

**Fallback Mode (Move):**
```
Bash(command: "mv '{vault_path}/{old_path}' '{vault_path}/{new_path}'")

# Update cross-references in other notes:
Grep(pattern: "\\[\\[old-name\\]\\]", path: "{vault_path}", type: "md")
# For each matching file:
Edit(file_path: "{matching_file}", old_string: "[[old-name]]", new_string: "[[new-name]]")
```

---

## Fallback Strategy (No MCP)

When the Obsidian MCP server is not available, SYNC mode operates in filesystem mode:

### Vault Discovery

Ask the user for the vault path if not known (same as READ mode fallback):
```
AskUserQuestion:
  question: "What is the path to your Obsidian vault?"
  header: "Vault path"
  options:
    - label: "~/obsidian-vault"
      description: "Common default (macOS/Linux)"
    - label: "~/Documents/Obsidian"
      description: "Documents folder (macOS/Linux)"
    - label: "C:\\Users\\{username}\\Documents\\Obsidian"
      description: "Documents folder (Windows)"
```

### Writing Notes (Fallback)

```
# Ensure destination directory exists
Bash(command: "mkdir -p {vault_path}/{destination}")

# Construct full file content (frontmatter + body)
# See YAML Serialization Pattern in Step 5 above

# Write the file
Write(file_path: "{vault_path}/{destination}/{filename}.md", content: "{frontmatter_yaml}\n\n{body}")
```

### Browsing Vault (Fallback)

```
# List vault structure
Glob(pattern: "{vault_path}/**/*.md")

# Or browse directories
Bash(command: "find {vault_path} -maxdepth 2 -type d")
```

### Patching Notes (Fallback)

```
Read(file_path: "{vault_path}/{note_path}")
Edit(file_path: "{vault_path}/{note_path}", old_string: "...", new_string: "...")
```

### Moving/Archiving Notes (Fallback)

```
Bash(command: "mv '{vault_path}/{old}' '{vault_path}/{new}'")
```

### Deleting Notes (Fallback)

```
# Always confirm with AskUserQuestion first
Bash(command: "rm '{vault_path}/{file}'")
```

### Limitations in Fallback Mode

| Feature | MCP Mode | Fallback Mode |
|---------|----------|---------------|
| Write notes | Full support | Full support (manual YAML serialization) |
| Browse vault | Full support | Full support (via Glob/Bash) |
| Patch notes | Atomic string replace | Read + Edit (functionally equivalent) |
| Move/rename | Native API | Bash mv (functionally equivalent) |
| Delete | Double confirmation | Bash rm with AskUserQuestion confirmation |
| Frontmatter serialization | Automatic (MCP handles) | Manual (construct YAML string) |
| Auto-create directories | Automatic | Bash mkdir -p |
| Cross-ref validation | Native API | Read + parse + Write (slower on large vaults) |

---

## Limitations

- **MCP recommended**: The Obsidian MCP server provides optimized operations. When unavailable, filesystem fallback provides full functionality with manual YAML serialization.
- **Markdown only**: This mode handles `.md` files. Binary assets, images, and non-text files are not synced
- **No bidirectional sync**: This mode writes TO Obsidian, it does not pull changes back from Obsidian
- **No conflict resolution**: If a note exists at the destination, it will be overwritten (with user confirmation)
- **Single vault**: Operates on the vault connected to the MCP server; cannot switch between multiple vaults

---

## Example: Full Sync Flow

**User request:** "sync {output_dir}/planning/ to obsidian"

1. **Glob**: Find all `.md` files in `{output_dir}/planning/`
2. **Read**: Read all files in parallel
3. **Load**: `ToolSearch query: "+obsidian write"`
4. **List vault**: `mcp__obsidian__list_directory(path: "/")`
5. **Ask user**: "Where should I save these 3 documents?" → User picks `work/agent-sync-sdk/plans`
6. **Write**: Sequential writes to `work/agent-sync-sdk/plans/00-strategic-analysis.md`, etc.
7. **Validate**: Check cross-references, fix missing bidirectional links
8. **Report**: "Synced 3 files to Obsidian: work/agent-sync-sdk/plans/"
