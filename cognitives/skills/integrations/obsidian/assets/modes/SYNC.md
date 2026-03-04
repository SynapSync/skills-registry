# SYNC Mode - Write to Obsidian Vault

## When to Use This Mode

- User asks to **sync**, **save**, **move**, or **store** files to Obsidian
- (ES) User says "guardar en obsidian", "sincronizar a obsidian", or similar
- After another skill finishes generating documents, user wants them in Obsidian
- User wants to **browse their vault** to choose where to place documents

## When NOT to Use This Mode

- User wants to read or search notes already in Obsidian (use READ mode)
- Files are not markdown or text-based (this mode is for `.md` content)

---

## Critical Rules

**RULE 0 - USE THE EXACT PATH THE USER GIVES YOU**

When the user specifies a vault destination path, use it **exactly as given**. Do not modify, substitute, or "improve" it.

**RULE 1 - ALWAYS ASK WHERE TO SAVE**

Never assume the destination folder. Always browse the vault and let the user pick or specify. If the user already specified a path, confirm it and use it directly.

**RULE 2 - PRESERVE CONTENT INTEGRITY**

- Read the source file completely before writing
- Never modify document body (headings, paragraphs, tables, code blocks, `## Referencias`)
- Only ADD or MERGE frontmatter metadata
- Cross-reference fixes update only the `related` frontmatter array

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

Infer values from:
- `title`: First H1 heading, or filename
- `date`: File modification date, or today
- `updated`: Today (sync date)
- `project`: Current working directory name or git repo name
- `type`: Infer from content using the 14-type taxonomy (see [obsidian-md-standard](../standards/obsidian-md-standard.md))
- `status`: `"active"` for new documents, preserve existing
- `tags`: Infer from project name, document type, and key topics
- `related`: Extract from `[[wiki-links]]` found in content, or from `## Referencias` section

**RULE 4 - REPORT RESULTS**

After syncing, always report: how many files were synced, exact vault paths, and any files skipped with reason.

---

## Obsidian Output Standard

When syncing, follow the [Obsidian markdown standard](../standards/obsidian-md-standard.md):

1. **Frontmatter enrichment**: Enrich minimal frontmatter with the full universal schema
2. **Frontmatter preservation**: Preserve all existing fields — only add missing ones
3. **Frontmatter generation**: Generate complete schema for documents with no frontmatter
4. **Wiki-link preservation**: Never convert `[[wiki-links]]` to `[markdown](links)`
5. **Type inference**: Map content to the 14-type taxonomy
6. **Cross-reference validation**: After batch sync, verify `related` fields are bidirectional

---

## Workflow

### Step 0: Resolve Vault Path

1. If vault path is known from conversation context or CLAUDE.md → use it
2. Otherwise ask:

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

Set `{vault_path}` — all subsequent steps use this as the base.

### Step 1: Identify What to Sync

Use `Glob` to discover source files based on user request (specific file, directory, or pattern).

### Step 2: Browse Vault and Get Destination

List vault structure and let the user choose where to save:

```
Glob(pattern: "{vault_path}/*/")
# Or for deeper browsing:
Bash(command: "find {vault_path} -maxdepth 2 -type d | head -30")
```

Present options via `AskUserQuestion`. Always include "Create new folder" option.

If the user already specified a destination path, skip browsing and use it directly.

### Step 3: Read Source Files

Read all source files in parallel using the `Read` tool. Check for existing frontmatter and extract metadata.

### Step 4: Write to Obsidian

For each file:

1. Generate frontmatter — see [frontmatter-generator.md](../helpers/frontmatter-generator.md)
2. Serialize to YAML — see **YAML Serialization Pattern** below
3. Construct full content: `---\n{yaml}\n---\n\n{body}`
4. Ensure destination directory exists:
   ```
   Bash(command: "mkdir -p {vault_path}/{destination}")
   ```
5. Write the file:
   ```
   Write(file_path: "{vault_path}/{destination}/{filename}.md", content: "{full_content}")
   ```

### Step 4a: Small Edits (Partial Updates)

When only a small change is needed to an existing note (e.g., updating a status field):

```
Read(file_path: "{vault_path}/{note_path}")
# Find the exact string to replace
Edit(file_path: "{vault_path}/{note_path}", old_string: "status: active", new_string: "status: completed")
```

Use `Edit` for 1-3 targeted changes. Use `Write` for full re-syncs or changes affecting >30% of the document.

### Step 5: Cross-Reference Validation

After syncing 2+ files, validate bidirectional references.

See [cross-ref-validator.md](../helpers/cross-ref-validator.md) for the complete workflow.

**Example output:**
```
Cross-reference validation:
- SPRINT-1.md references [[PROGRESS]] ✓ (PROGRESS references back)
- ANALYSIS.md references [[CONVENTIONS]] ✓ (CONVENTIONS references back)
- PLANNING.md references [[ANALYSIS]] ⚠ Fixed: added [[PLANNING]] to ANALYSIS related field
```

### Step 6: Report Results

```
Synced to Obsidian:
- work/my-project/plans/00-strategic-analysis.md
- work/my-project/plans/01-technical-debt.md

2 files synced to: work/my-project/plans/
```

---

## YAML Serialization Pattern

When writing frontmatter, construct YAML manually:

Given frontmatter object:
```json
{
  "title": "Document Title",
  "date": "2026-02-10",
  "updated": "2026-03-03",
  "project": "my-project",
  "type": "analysis",
  "status": "active",
  "version": "1.0",
  "tags": ["tag1", "tag2"],
  "changelog": [{"version": "1.0", "date": "2026-03-03", "changes": ["Synced to Obsidian"]}],
  "related": ["[[doc-a]]", "[[doc-b]]"]
}
```

Serialize as:
```yaml
---
title: "Document Title"
date: "2026-02-10"
updated: "2026-03-03"
project: "my-project"
type: "analysis"
status: "active"
version: "1.0"
tags:
  - tag1
  - tag2
changelog:
  - version: "1.0"
    date: "2026-03-03"
    changes:
      - "Synced to Obsidian"
related:
  - "[[doc-a]]"
  - "[[doc-b]]"
---
```

Then prepend to body: `{yaml_block}\n\n{body_content}`

---

## Batch Sync Pattern

When syncing an entire folder, process files efficiently:

See [batch-sync-pattern.md](../helpers/batch-sync-pattern.md) for the full workflow.

**Key principles:**
1. `Glob` all `.md` files once
2. `Read` all files in parallel
3. `Bash mkdir -p` to ensure destination exists
4. Ask user once for destination
5. `Write` all files sequentially
6. Validate cross-references after all writes
7. Report all results at the end

---

## Optional Workflow: Archive and Delete

> **DELETE is destructive and irreversible.** Always confirm before deleting.

### Archive (Preferred)

```
Bash(command: "mkdir -p {vault_path}/archive/{project}")
Bash(command: "mv '{vault_path}/{old_path}' '{vault_path}/archive/{project}/{filename}'")
# Update frontmatter status to archived:
Read(file_path: "{vault_path}/archive/{project}/{filename}")
Edit(file_path: "{vault_path}/archive/{project}/{filename}", old_string: "status: active", new_string: "status: archived")
```

### Delete (Destructive)

1. Confirm with user via `AskUserQuestion`:
   ```
   question: "Are you sure you want to permanently delete '{filename}'? This cannot be undone."
   options:
     - "Yes, delete permanently"
     - "No, move to archive instead"
   ```
2. If confirmed:
   ```
   Bash(command: "rm '{vault_path}/{file_path}'")
   ```

---

## Optional Workflow: Move and Reorganize

### Single Note Move

```
Bash(command: "mv '{vault_path}/{old_path}' '{vault_path}/{new_path}'")
```

### Batch Reorganization

1. List source directory: `Glob(pattern: "{vault_path}/{folder}/**/*.md")`
2. Ask user for destination mapping
3. Move files: `Bash mv` for each
4. Update cross-references if filenames changed:
   ```
   Grep(pattern: "\\[\\[old-name\\]\\]", path: "{vault_path}", type: "md")
   # For each matching file:
   Edit(file_path: "{matching_file}", old_string: "[[old-name]]", new_string: "[[new-name]]")
   ```
5. Report all moves

---

## Output Format Contract

Every SYNC operation MUST produce a response with these sections:

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
- [any overwrites, missing sources — or "None"]
```

---

## Best Practices

- **Before Sync**: Verify source files exist and producer skill has completed
- **During Sync**: Never modify content (only frontmatter), process writes sequentially, warn before overwriting
- **After Sync**: Report all synced paths, any failures, and suggest visual verification in Obsidian
