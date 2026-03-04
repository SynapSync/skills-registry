# READ Mode - Read from Obsidian Vault

## When to Use This Mode

- User asks to **read, consult, or check** notes in Obsidian
- User asks "what do my notes say about X?" or "check my plans in obsidian"
- User needs **project context** that lives in Obsidian (plans, decisions, architecture docs)
- User asks to **search** for specific topics, decisions, or information across the vault
- User wants to **use Obsidian notes as context** for a current task or decision
- Another skill or agent needs knowledge that resides in the vault
- (ES) User says "lee de obsidian", "busca en mis notas", "consulta obsidian"

## When NOT to Use This Mode

- User wants to **write** to Obsidian (use SYNC mode instead)
- The information is already in the current workspace (use Read/Grep/Glob directly)
- User asks about files that are clearly local to the project, not in the vault

---

## Critical Rules

**RULE 1 - NEVER FABRICATE CONTENT**

Only report information that exists verbatim in the notes:
- Quote relevant passages with the source note path
- Clearly distinguish between "the note says X" and "based on the notes, I interpret X"
- If information is not found, say so explicitly rather than guessing

**RULE 2 - RESPECT VAULT STRUCTURE**

Obsidian vaults have semantic meaning in their structure:
- **Folders** = categories/projects/topics
- **Tags** (in frontmatter or inline `#tag`) = cross-cutting classification
- **Wikilinks** (`[[note-name]]`) = relationships between concepts
- **Frontmatter** = structured metadata (date, type, project, status)

Read and interpret all of these, not just the body text.

**RULE 3 - PRIORITIZE BY RELEVANCE**

When retrieving information, rank by:
1. **Active/current** over historical (check `date` frontmatter, prefer recent)
2. **Directly referenced** over tangentially related
3. **Plans and decisions** over raw notes (check `type` frontmatter)
4. **Tagged/structured** over unstructured notes

**RULE 4 - READ MODE IS STRICTLY READ-ONLY**

Never write to the vault in this mode. If compliance issues are found, report them — do not fix them. Fixes require SYNC mode.

**RULE 5 - STRUCTURED OUTPUT FOR COMPOSABILITY**

When another skill or agent requests knowledge, return structured summaries:
- Source note path
- Key findings
- Relevant quotes
- Confidence level (exact match vs. inferred)
- Related notes that may contain additional context

---

## Capabilities

### Reading
- Read individual notes by path
- Read all notes in a folder (recursive)
- Read notes matching a pattern (glob)
- Parse and extract frontmatter metadata
- Extract inline tags (`#tag`)
- Detect and follow wikilinks (`[[note]]`)

### Searching
- Full-text search across the vault (Grep)
- Filter by frontmatter fields (project, type, date, tags)
- Filter by folder/path
- Search by tag combinations
- Find notes linking to a specific note (backlinks via Grep)

### Reasoning
- Summarize content from multiple notes
- Correlate information across documents
- Answer questions using vault content as source
- Identify the most recent/relevant version of a topic
- Build a context snapshot for the current task

### Discovery
- Map vault structure (folder tree)
- List projects and their documents
- Identify recently modified notes

### Standard Compliance Check
- Report which documents follow the Obsidian markdown standard
- Identify documents missing required frontmatter fields
- Flag one-directional references
- List documents without `## Referencias` sections

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

Set `{vault_path}` — all subsequent operations use this as the base.

### Step 1: Understand the Request

Parse the user's intent into one of these operations:

| User Intent | Operation |
|-------------|-----------|
| Read specific note | `READ_NOTE` |
| Read a folder | `READ_FOLDER` |
| Search by topic | `SEARCH_TEXT` |
| Search by tag | `SEARCH_TAG` |
| Search by metadata | `SEARCH_META` |
| Get project context | `PROJECT_CONTEXT` |
| Answer a question | `REASON` |
| Explore structure | `DISCOVER` |
| Check standard compliance | `COMPLIANCE_CHECK` |

### Step 2: Execute the Operation

#### READ_NOTE

```
Read(file_path: "{vault_path}/{note_path}.md")
```

After reading:
1. Parse frontmatter (title, date, project, type, tags)
2. Extract wikilinks (`[[referenced-note]]`)
3. Extract inline tags (`#tag-name`)
4. Present structured summary + full content if requested

#### READ_FOLDER

```
Glob(pattern: "{vault_path}/{folder}/**/*.md")
# Then Read each file in parallel
```

After reading all notes:
1. Sort by date (frontmatter `date` field or filename)
2. Build table of contents with title + type + date
3. Summarize each note in 1-2 sentences
4. Identify relationships between notes (shared tags, wikilinks)

#### SEARCH_TEXT

```
Grep(pattern: "search term", path: "{vault_path}", type: "md", -i: true)
# Then Read matching files for context
```

For case-insensitive search, use `-i: true`. For exact phrases, quote the pattern.

Present results as:
```
Found N notes matching "query":

1. work/my-project/plans/01-technical-debt.md
   > "El LockFileManager realiza operaciones sin mutex..."
   Type: technical-debt | Date: 2026-02-10
```

#### SEARCH_TAG

```
# Search inline tags
Grep(pattern: "#strategy", path: "{vault_path}", type: "md")
# Search frontmatter tags
Grep(pattern: "tags:.*strategy", path: "{vault_path}", type: "md")
```

#### SEARCH_META

```
# Find all notes for a specific project
Grep(pattern: "project: my-project", path: "{vault_path}", type: "md")

# Find all notes of a specific type
Grep(pattern: "type: technical-debt", path: "{vault_path}", type: "md")

# Find notes from a date range
Grep(pattern: "date: \"2026-02", path: "{vault_path}", type: "md")
```

#### PROJECT_CONTEXT

Assembles a comprehensive context snapshot for a project:

1. **Locate project folder**:
   ```
   Glob(pattern: "{vault_path}/**/")
   # Find folder matching project name
   ```

2. **Collect all notes**:
   ```
   Glob(pattern: "{vault_path}/{project_folder}/**/*.md")
   ```

3. **Read all notes** in parallel — parse frontmatter, extract type, status, date

4. **Sort and classify** by type:
   - Plans & roadmaps (type: plan, strategic-analysis)
   - Technical debt (type: technical-debt)
   - Architecture (type: architecture)
   - Sprint plans (type: sprint-plan)
   - Reports (type: analysis)

5. **Build context summary**:
   ```
   ## Project Context: my-project
   Last updated: 2026-02-10

   ### Active Plans
   - 00-strategic-analysis.md: Roadmap Q1-Q4 2026, 32 action items

   ### Known Issues
   - 01-technical-debt.md: 18 findings (5 critical, 8 medium, 5 low)

   ### Key Decisions
   - [extracted from notes with type: decision-log]

   ### Related Notes
   - [notes in other folders linking to this project]
   ```

#### REASON

Answer a question using vault content as source:

1. Identify relevant notes using SEARCH_TEXT + SEARCH_TAG + SEARCH_META
2. Read the top 3-7 most relevant notes
3. Extract relevant passages
4. Synthesize an answer with citations:
   ```
   Based on your Obsidian notes:

   [Answer with reasoning]

   Sources:
   - work/my-project/plans/00-strategic-analysis.md (Section 5.1)
   - work/my-project/plans/01-technical-debt.md (Section 3.1)
   ```

#### DISCOVER

Map vault structure:

```
Bash(command: "find {vault_path} -maxdepth 2 -type d")
Glob(pattern: "{vault_path}/**/*.md")
```

Count notes per folder, identify recently modified:
```
Bash(command: "find {vault_path} -name '*.md' -newer {vault_path} -maxdepth 3 | head -10")
```

Present as:
```
Your Obsidian Vault:

work/
  my-project/
    plans/     (3 notes - last updated 2026-02-10)
  other-project/
    ...

Stats: X total notes, Y folders
Recently modified: [list of 5-10 most recent]
```

#### COMPLIANCE_CHECK

Analyze documents against the Obsidian markdown standard.

> READ mode only **diagnoses** — never writes or fixes. Switch to SYNC mode to apply fixes.

See [../validators/obsidian-linter.md](../validators/obsidian-linter.md) for complete validation rules.

1. List all `.md` files: `Glob(pattern: "{vault_path}/{project}/**/*.md")`
2. Read each file and parse frontmatter
3. Validate: required fields, wiki-link syntax, tag format, bidirectional refs, type taxonomy, `## Referencias` section
4. Generate compliance report (see output format below)

### Step 3: Present Results

#### READ_NOTE / READ_FOLDER output:

```markdown
### Note: {title}
| Field | Value |
|-------|-------|
| Path | `{vault_path}` |
| Type | {type} |
| Date | {date} |
| Status | {status} |
| Tags | {tags} |

{content — full or summarized if >500 lines}

**Outgoing links:** [[link1]], [[link2]]
```

#### SEARCH output:

```markdown
### Search Results: "{query}"
Found {N} notes:

1. **{vault_path}**
   Type: {type} | Date: {date}
   > "{quoted passage}"
```

#### COMPLIANCE_CHECK output:

```markdown
### Compliance Report: {project}
Date: {today}

**Summary:** {total} documents, {compliant} compliant, {partial} partial, {non-compliant} non-compliant

| Document | Issue | Severity |
|----------|-------|----------|
| {filename} | Missing `updated` field | Low |
| {filename} | No `## Referencias` section | Medium |

### Recommendations
1. {specific actionable fix}
```

---

## Frontmatter Parsing

Expected frontmatter schema for reading/filtering:

```yaml
---
title: "Document Title"
date: "2026-02-10"
updated: "2026-02-11"
project: "my-project"
type: "strategic-analysis"
status: "active"
version: "1.2"
tags: [strategy, roadmap, plan]
related:
  - "[[01-technical-debt]]"
  - "[[02-growth-vision]]"
---
```

When reading frontmatter manually, parse the YAML block between the `---` delimiters.

---

## Type Taxonomy

| Type | Priority | Description |
|------|----------|-------------|
| `analysis` | High | General analysis documents |
| `plan` | High | Strategic planning |
| `execution-plan` | High | Concrete task breakdowns |
| `sprint-plan` | High | Sprint-level task plans |
| `progress` | High | Master progress dashboards |
| `conventions` | Medium | Project patterns and conventions |
| `requirements` | Medium | Functional/non-functional requirements |
| `architecture` | Medium | System design and decisions |
| `technical-report` | Medium | Code/module analysis reports |
| `refactor-plan` | Medium | Refactoring recommendations |
| `retrospective` | Medium | Sprint/project retrospectives |
| `decision-log` | Medium | Architecture/engineering decisions |
| `data-model` | Low | Entity relationships |
| `flow-diagram` | Low | Core flows and sequences |

---

## Relationship Mapping

### Related Field (Primary Source)

```yaml
related:
  - "[[01-technical-debt]]"
  - "[[02-growth-vision]]"
```

When building relationship maps:
1. **First**: Check `related` in frontmatter — explicit, curated relationships
2. **Second**: Check `## Referencias` section — structured navigation links
3. **Third**: Check wikilinks in body text — implicit references
4. **Fourth**: Check backlinks — notes that reference this one

### Backlinks

Find notes that link TO a specific note:

```
Grep(pattern: "\\[\\[note-name\\]\\]", path: "{vault_path}", type: "md")
```

---

## Integration with Other Skills

| Skill | How READ mode integrates |
|-------|--------------------------|
| `obsidian` SYNC mode | Read notes previously synced; verify sync status |
| `universal-planner` | Provide existing project context and historical plans |
| `code-analyzer` | Supply architecture notes as reference for analysis |

---

## Best Practices

- Scope searches to relevant folders — don't scan the entire vault for project-specific queries
- Parse frontmatter on every note read — it's the primary metadata source
- For large notes (>500 lines), summarize instead of presenting full content unless asked
- Always cite sources with exact note paths
- Distinguish quoted content from your interpretation
- If information seems outdated (old `date` field), warn the user
- Avoid reading more than 10 full notes in a single operation unless explicitly requested

---

## Limitations

- **Read-only**: Does not modify vault content (use SYNC mode for writing)
- **No real-time sync**: Reads vault state at query time
- **Markdown only**: Processes `.md` files; ignores images, PDFs, and other attachments
- **Grep-based search**: Literal/regex only — no fuzzy matching. Broaden the pattern if results are sparse.
- **Single vault**: Operates on one vault at a time
- **Context window**: Very large vaults require targeted queries; reading everything is not feasible
