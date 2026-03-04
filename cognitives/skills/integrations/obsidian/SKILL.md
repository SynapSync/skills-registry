---
name: obsidian
description: >
  Unified Obsidian vault operations: sync documents to vault, read notes for context,
  search knowledge, and validate markdown standards. Filesystem-based, no MCP required.
  Trigger: When user wants to read from or write to Obsidian vault.
license: Apache-2.0
metadata:
  author: synapsync
  version: "4.0"
  scope: [root]
  auto_invoke:
    # English — SYNC
    - "sync * to obsidian"
    - "save * to obsidian"
    - "write * to obsidian"
    - "move * to obsidian"
    - "send * to obsidian"
    - "sync * to vault"
    - "save * to vault"
    - "write * to vault"
    - "move * to vault"
    - "send * to vault"
    - "sync the output to *"
    # English — READ
    - "read obsidian notes about *"
    - "search my vault for *"
    - "check obsidian for *"
    - "read from vault *"
    - "search obsidian for *"
    - "what do my notes say about *"
    # Spanish — SYNC
    - "guardar en obsidian"
    - "sincronizar a obsidian"
    - "manda * a obsidian"
    - "guardar en el vault"
    - "enviar a obsidian"
    - "sincronizar al vault"
    # Spanish — READ
    - "lee de obsidian"
    - "busca en obsidian"
    - "busca en el vault"
    - "consulta en obsidian"
    - "lee del vault"
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, AskUserQuestion
---

# Obsidian Vault Manager

## Assets

This skill uses a modular assets architecture. Detailed workflows, templates, and helpers are in the [assets/](assets/) directory:

- **[assets/modes/](assets/modes/)** - 2 operation modes with detailed workflows
- **[assets/helpers/](assets/helpers/)** - Shared helpers for frontmatter, cross-refs, ranking, batch operations

See [assets/README.md](assets/README.md) for the directory index.

> **Metadata note**: This file's YAML frontmatter defines runtime behavior (auto_invoke triggers, allowed-tools, scope). The `manifest.json` defines registry metadata (version, tags, providers, dependencies). The frontmatter is consumed by the Claude Code skill loader; the manifest is consumed by the `skills` CLI for installation and updates.

---

## Purpose

Unified skill for all Obsidian vault operations. Acts as a **bidirectional knowledge bridge** between the agent workspace and your Obsidian vault (MCP preferred, filesystem fallback when unavailable):

- **SYNC mode**: Write documents from workspace to vault (markdown reports, plans, analysis)
- **READ mode**: Read, search, and reason over vault notes as contextual knowledge source

Combines the capabilities of the former `obsidian-sync` and `obsidian-reader` skills into one cohesive tool with progressive disclosure.

---

## Language Support (i18n)

This skill supports **bilingual operation** (English + Spanish):

- **Trigger keywords**: Both English (`sync`, `read`, `search`) and Spanish (`guardar`, `lee`, `busca`) are recognized for mode detection.
- **`## Referencias` section header**: Uses the Spanish form intentionally as an Obsidian convention adopted by this project. All documents produced by this skill use `## Referencias` (not `## References`).
- **Example queries**: Documentation includes examples in both languages to reflect real bilingual usage.
- **Output language**: The skill responds in the **same language the user uses**. If the user writes in Spanish, respond in Spanish. If in English, respond in English. Document content is never translated — it is synced/read exactly as-is.

> **Convention**: `## Referencias` is a fixed section header. Never translate it to `## References` in generated documents.

---

## Critical Rules

> **RULE 1 — MODE DETECTION: AUTO-SELECT FROM USER INTENT**
>
> Detect the operation mode from the user's input. Never force a mode that doesn't match the scenario.

> **RULE 2 — PRESERVE CONTENT INTEGRITY (SYNC mode)**
>
> Read source files completely before writing. Never modify document body content (headings, paragraphs, tables, code blocks, `## Referencias` section). Only add/merge frontmatter metadata. Cross-reference fixes update the `related` frontmatter array only — never the document body.

> **RULE 3 — NEVER FABRICATE CONTENT (READ mode)**
>
> Only report information that exists verbatim in notes. Quote sources with paths. Distinguish between "the note says X" and "I interpret X based on notes".

> **RULE 4 — FOLLOW OBSIDIAN MARKDOWN STANDARD**
>
> All operations follow the [Obsidian markdown standard](assets/standards/obsidian-md-standard.md) specification for frontmatter, wiki-links, types, and cross-references.

---

## Tool Dependencies

This skill uses native Claude Code tools only: `Read`, `Write`, `Edit`, `Glob`, `Grep`, `Bash`, `AskUserQuestion`. No MCP or external dependencies required.

---

## Operation Modes

### Mode Detection

Auto-detect from user intent using these signals:

| Mode | Signals (EN) | Signals (ES) | Example Inputs |
|------|-------------|-------------|----------------|
| **SYNC** | "sync", "save", "store", "write", "move" + "obsidian" | "guardar", "sincronizar" + "obsidian" | "sync this report to obsidian", "guardar en obsidian" |
| **READ** | "read", "search", "find", "check", "consult", "what do my notes say" + "obsidian"/"vault"/"notes" | "lee", "busca", "consulta" + "obsidian"/"vault"/"notas" | "read my vault notes about X", "busca en obsidian" |

**Ambiguous intent resolution:** If the user's intent does not clearly match SYNC or READ signals (e.g., "help me with obsidian", "manage my notes", "obsidian" with no verb), ask the user to clarify:

```
AskUserQuestion:
  question: "What would you like to do with your Obsidian vault?"
  header: "Mode"
  options:
    - label: "Read / Search"
      description: "Read notes, search for information, or get project context from your vault"
    - label: "Sync / Write"
      description: "Save or sync documents from your workspace to your vault"
```

### Mode Capabilities Matrix

| Capability | SYNC Mode | READ Mode |
|-----------|:---------:|:---------:|
| Write to vault | ✅ | ❌ |
| Read from vault | ❌ | ✅ |
| Search vault | ❌ | ✅ |
| Generate frontmatter | ✅ | ❌ |
| Cross-ref fix (frontmatter) | ✅ | ❌ |
| Cross-ref diagnose | ✅ | ✅ |
| Batch operations | ✅ | ✅ |
| Vault browsing | ✅ | ✅ |
| Priority ranking | ❌ | ✅ |
| Standard compliance check | ❌ | ✅ |
| Delete/archive notes | ✅ | ❌ |
| Move/reorganize notes | ✅ | ❌ |
| Patch notes (partial edit) | ✅ | ❌ |
| Metadata-only reads | ❌ | ✅ |

---

## Asset Loading (Mode-Gated)

After detecting the mode, read ONLY the assets listed for that mode. Do NOT read assets for other modes — they waste context tokens.

| Mode | Read These Assets | Do NOT Read |
|------|-------------------|-------------|
| **SYNC** | `assets/modes/SYNC.md` | READ.md, priority-ranking.md, obsidian-linter.md |
| **READ** | `assets/modes/READ.md` | SYNC.md, frontmatter-generator.md, cross-ref-validator.md, batch-sync-pattern.md |

Each mode asset references its required helpers and standards internally. Read them on-demand as the mode workflow instructs — not upfront.

For MCP tool parameter contracts (needed in both modes), see [assets/helpers/tool-reference.md](assets/helpers/tool-reference.md) — read only if ToolSearch output is insufficient.

---

## Quick Start

### SYNC Mode

Use to save documents from workspace to your Obsidian vault:

> Sync this report to obsidian.

**Assets to read now:** [assets/modes/SYNC.md](assets/modes/SYNC.md) (references frontmatter-generator, cross-ref-validator, batch-sync-pattern on-demand)

### READ Mode

Use to read, search, and reason over your vault notes:

> Read my vault notes about the architecture.

**Assets to read now:** [assets/modes/READ.md](assets/modes/READ.md) (references priority-ranking, obsidian-linter on-demand)

---

## Mode Workflows

### SYNC Mode - Write to Obsidian Vault

Sync markdown documents from workspace to Obsidian vault with proper frontmatter and cross-reference validation.

**Assets to read now:** [assets/modes/SYNC.md](assets/modes/SYNC.md) (references frontmatter-generator, cross-ref-validator, batch-sync-pattern on-demand)

**Quick summary:**
1. Identify source files (Glob workspace docs)
2. Detect access mode (MCP or filesystem fallback)
3. Browse vault and ask user for destination
4. Read source files
5. Generate frontmatter (→ SYNC.md references [frontmatter-generator](assets/helpers/frontmatter-generator.md))
6. Write to vault with `mcp__obsidian__write_note`
7. Validate cross-references (→ SYNC.md references [cross-ref-validator](assets/helpers/cross-ref-validator.md))
8. Report results

### READ Mode - Read from Obsidian Vault

Read, search, and reason over vault notes to provide contextual knowledge for decision-making.

**Assets to read now:** [assets/modes/READ.md](assets/modes/READ.md) (references priority-ranking, obsidian-linter on-demand)

**Quick summary:**
1. Load MCP tools (fallback to filesystem if MCP unavailable)
2. Parse user intent (read note, search, get project context, answer question)
3. Execute operation (search/read/reason)
4. Rank results by priority (→ READ.md references [priority-ranking](assets/helpers/priority-ranking.md))
5. Present structured output with source citations

---

## Configuration Resolution

`{vault_destination}` is resolved at runtime — SYNC mode browses the vault and asks the user where to save. No pre-configuration or persistence needed. See SYNC.md Step 3.

**Staging-aware**: When the user says "sync my output to vault", SYNC mode detects `.agents/` directories from producer skills (universal-planner, code-analyzer), lists available output, lets the user pick, and syncs to the chosen vault destination.

---

## Shared Helpers

- **[frontmatter-generator](assets/helpers/frontmatter-generator.md)** - Generate Obsidian frontmatter following universal schema
- **[cross-ref-validator](assets/helpers/cross-ref-validator.md)** - Validate bidirectional references
- **[batch-sync-pattern](assets/helpers/batch-sync-pattern.md)** - Optimized batch file sync workflow
- **[priority-ranking](assets/helpers/priority-ranking.md)** - Rank search results by relevance

---

## Obsidian Output Standard

All operations follow the internal [Obsidian markdown standard](assets/standards/obsidian-md-standard.md):

1. **Frontmatter**: Universal schema with title, date, project, type, status, version, tags, changelog, related
2. **Type taxonomy**: 14 document types (analysis, plan, sprint-plan, technical-report, etc.)
3. **Wiki-links**: `[[note-name]]` format, never markdown links
4. **Bidirectional refs**: If A→B, then B→A
5. **Status transitions**: draft → active → completed → archived

For compliance validation, see [assets/validators/obsidian-linter.md](assets/validators/obsidian-linter.md).

---

## Integration with Other Skills

| Producer Skill | How obsidian Integrates |
|---------------|------------------------|
| `universal-planner` | SYNC: Saves planning docs to vault. READ: Provides historical plans as context |
| `code-analyzer` | SYNC: Saves technical reports. READ: Surfaces architecture notes |
| `universal-planner` (EXECUTE mode) | READ: Retrieves sprint plans and progress |

**Composition pattern:**
```
Producer skill generates docs → obsidian SYNC mode writes to vault
Agent needs context → obsidian READ mode retrieves from vault
```

## Cross-Skill Invocation

When another skill or agent needs to invoke obsidian, use this priority order:

### Priority 1: Skill Tool (recommended)

If running in Claude Code with skills loaded:

```
Skill("obsidian")
```

Then describe the operation (e.g., "sync the files in .agents/staging/code-analyzer/ to the vault").

### Priority 2: Natural Language Trigger

Say a phrase that matches auto_invoke:

- "Sync the output to obsidian"
- "Save these reports to the vault"
- "Guardar en obsidian"

### Priority 3: Direct SKILL.md Read (subagents without skill context)

If the Skill tool is unavailable (e.g., team subagents via Task tool), read the SKILL.md directly:

```
Read: cognitives/skills/integrations/obsidian/SKILL.md
```

Then follow the SYNC or READ mode workflow. NEVER call `mcp__obsidian__*` tools without reading this skill first.

---

## Best Practices

**SYNC mode:**
- Verify source files exist before syncing
- Always ask user where to save (never assume destination)
- Validate cross-references after batch syncs
- Report exact vault paths where files were saved

**READ mode:**
- Try MCP first, fallback to filesystem if unavailable
- Scope searches to relevant folders (don't search entire vault unnecessarily)
- Always cite sources with exact note paths
- Distinguish quoted content from interpretation

---

## Limitations

- **Markdown only**: Handles `.md` files, not binary assets/images
- **Single vault**: Operates on one vault at a time
- **No bidirectional sync**: Writes TO vault (SYNC) or reads FROM vault (READ), not both-way sync
- **Read-only in READ mode**: Cannot modify notes in READ mode (use SYNC for writing)
- **Search is regex/literal**: Uses Grep — no fuzzy search. Broaden the pattern if results are sparse.

---

## Troubleshooting

| Issue | Mode | Solution |
|-------|------|----------|
| "Tool not found: mcp__obsidian__*" | Both | Run `ToolSearch query: "+obsidian write"` first |
| "MCP server not connected" | Both | Skill automatically falls back to filesystem mode. Provide vault path when prompted. |
| "Note already exists" | SYNC | Confirm with user before overwriting |
| "No results found" | READ | Broaden search, check different folders, verify vault path |
| "Empty content" | SYNC | Verify source file path with Glob |
| "Confirmation path mismatch" | SYNC | `delete_note` requires `confirmPath` to exactly match `path` |
| "Multiple matches found" | SYNC | `patch_note` with `replaceAll: false` fails on multiple matches; set `replaceAll: true` or refine the string |
| "Frontmatter merge conflict" | SYNC | `update_frontmatter` with `merge: true` preserves existing; use `merge: false` to replace entirely |
| Slow metadata queries | READ | Use `get_frontmatter` or `get_notes_info` instead of reading full notes |
