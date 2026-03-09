# CLAUDE.md - SynapSync Registry

## Project Overview

This is the **SynapSync Registry** — the official public registry of cognitives (skills, agents, prompts, workflows, tools). It is a content-only repository with no build system, no dependencies, and no executable code.

Cognitives are reusable AI instructions that help AI assistants understand project patterns and conventions. They work across multiple providers (Claude, OpenAI, Cursor, Windsurf, Copilot, Gemini) and are installed via the `skills` CLI:

```bash
npx skills add owner/repo    # Install cognitives from a registry
npx skills update             # Update installed cognitives
```

## Repository Structure

```
synapse-registry/
├── registry.json                        # Central index of all cognitives (source of truth)
├── cognitives/                          # Public registry content (downloadable)
│   ├── skills/{category}/{name}/        # AI instruction skills (SKILL.md + manifest.json)
│   ├── agents/{category}/{name}/        # Autonomous AI agents (AGENT.md + manifest.json)
│   ├── prompts/{category}/{name}/       # Reusable prompt templates (PROMPT.md + manifest.json)
│   ├── workflows/{category}/{name}/     # Multi-step processes (WORKFLOW.yaml + manifest.json)
│   └── tools/{category}/{name}/         # External integrations (TOOL.md + manifest.json)
└── core/                                # Internal project tooling (not published)
    └── register/                        # Cognitive registration automation
```

Every cognitive lives in `cognitives/{type}s/{category}/{name}/` and must contain:
1. `manifest.json` — metadata, version, author, tags, provider compatibility
2. A content file — `SKILL.md`, `AGENT.md`, `PROMPT.md`, `WORKFLOW.yaml`, or `TOOL.md`

## Current Cognitives

### Skills
- **skill-creator** (`cognitives/skills/general/skill-creator/`) — Meta-skill for creating new skills with templates. Includes `assets/` with basic and advanced templates.
- **universal-planner** (`cognitives/skills/planning/universal-planner/`) — Unified planning and execution skill with PLAN and EXECUTE modes. Covers new projects, features, refactors, bug fixes, tech debt, and architecture changes. Uses modular assets pattern with 8 modes (2 top-level + 6 planning sub-modes).
- **growth-architect** (`cognitives/skills/planning/growth-architect/`) — AI Co-Founder & Growth Architect: strategic clarity, product vision, MVP design, and ADRs. Single ANALYZE mode with optional focus modifiers (CTO, Product, Growth, Investor, Technical, Brutal). Modular assets: 1 mode, 1 helper, 2 templates.
- **code-analyzer** (`cognitives/skills/analytics/code-analyzer/`) — Analyzes code modules and generates structured technical reports with architecture diagrams.
- **obsidian** (`cognitives/skills/integrations/obsidian/`) — Unified Obsidian vault manager with SYNC and READ modes. Syncs documents to vault and reads/searches notes via filesystem (Read/Write/Edit/Glob/Grep). Includes the Obsidian markdown standard specification and linter as internal assets. Claude-only.
- **sprint-forge** (`cognitives/skills/workflow/sprint-forge/`) — Adaptive sprint workflow with 3 modes (INIT, SPRINT, STATUS). Deep analysis, evolving roadmap, one-at-a-time sprints, formal debt tracking, and re-entry prompts for context persistence. Language-agnostic. Modular assets: 3 modes, 4 helpers, 4 templates.
- **project-brain** (`cognitives/skills/workflow/project-brain/`) — Session memory for AI agents: LOAD mode recovers context, SAVE mode persists sessions. Auto-discovery of brain documents, incremental merge, configurable brain directory.
- **growth-ceo** (`cognitives/skills/planning/growth-ceo/`) — CEO-level strategic initiative engine. Generates high-leverage initiatives, product ideas, and growth opportunities across three time horizons. Conversational output, no file I/O.

### Agents
- **feature-branch-manager** (`cognitives/agents/general/feature-branch-manager/`) — Git workflow agent for branch creation, pushing, and PR creation. Claude-only.

## Internal Tooling (not published)

- **cognitive-register** (`core/register/`) — Automates registration of new cognitives into the registry. Used internally by contributors.

## Valid Categories

`general`, `frontend`, `backend`, `database`, `devops`, `security`, `testing`, `analytics`, `automation`, `integrations`, `planning`, `workflow`

## Valid Providers

`claude`, `openai`, `cursor`, `windsurf`, `copilot`, `gemini`, `codex`

## Cognitive Types

| Type | Content File | Description |
|------|-------------|-------------|
| skill | `SKILL.md` | Reusable AI instructions with YAML frontmatter |
| agent | `AGENT.md` or `{name}.md` | Autonomous AI entities with specific behaviors |
| prompt | `PROMPT.md` | Reusable prompts with variable substitution |
| workflow | `WORKFLOW.yaml` | Multi-step processes combining multiple cognitives |
| tool | `TOOL.md` | External API integrations |

## Key Conventions

### manifest.json Schema

```json
{
  "$schema": "https://synapsync.dev/schemas/cognitive-manifest.json",
  "name": "cognitive-name",
  "type": "skill|agent|prompt|workflow|tool",
  "version": "semver",
  "description": "Max 100 characters",
  "author": { "name": "", "url": "", "email": "" },
  "license": "Apache-2.0",
  "category": "one of the valid categories",
  "tags": ["max", "10", "tags"],
  "providers": ["claude", "openai"],
  "file": "SKILL.md",
  "repository": { "type": "git", "url": "" },
  "createdAt": "ISO 8601",
  "updatedAt": "ISO 8601"
}
```

### registry.json

The central index at the root. When adding a cognitive, an entry must also be added here with: `name`, `type`, `version`, `description`, `author`, `category`, `tags`, `providers`, `downloads` (starts at 0), and `path`.

### Naming Conventions

- Use kebab-case for cognitive names: `react-hooks`, `api-error-handling`
- Names should be descriptive, not generic (`utils`, `helpers` are bad)
- Pattern: `{technology}`, `{tech}-{feature}`, or `{framework}-{component}`

### SKILL.md Frontmatter

Skills use YAML frontmatter with: `name`, `description`, `license`, `metadata` (author, version, scope, auto_invoke), and `allowed-tools`.

**Important**: Do NOT put `output_dir` in frontmatter. Output paths are resolved at runtime via the Configuration Resolution convention (see below).

### Lazy Asset Loading Convention

Modular skills (2+ modes) MUST include mode-gated asset loading instructions in SKILL.md to prevent agents from reading unnecessary assets.

**Required pattern**: After `## Mode Detection`, add `## Asset Loading (Mode-Gated)` with a table:

| Mode | Read These Assets | Do NOT Read |
|------|-------------------|-------------|
| **MODE_A** | `mode-a.md`, `shared.md` | mode-b.md, b-only.md |

**Rules:**
- "Do NOT Read" column is mandatory — explicit exclusion is more effective than omission
- Quick Start uses "**Assets to read now:**" (imperative) instead of "See [file]" (passive reference)
- Mode-specific helpers are referenced from within the mode asset, not from SKILL.md
- `changelog:` key MUST NOT appear in SKILL.md frontmatter (canonical source: `manifest.json`)
- `## Version History` section MUST NOT appear in SKILL.md body

### Configuration Resolution Convention

Skills that produce output resolve `{output_dir}` at runtime — no external config files needed.

**Resolution algorithm** (same for all skills):

1. **User message context** — If the user's message contains file paths, extract `{output_dir}` from those paths
2. **Auto-discover** — Scan for `.agents/{skill-name}/` in `{cwd}`
3. **Ask the user** — If nothing found, ask where to save documents. Default suggestion: `.agents/{skill-name}/{project-name}/`

No AGENTS.md. No branded blocks. Each skill resolves its output directory at runtime.

**When creating a new skill that produces output**, you MUST:
1. Use `{output_dir}` for all output paths (e.g., `{output_dir}/planning/`, `{output_dir}/technical/`)
2. Include a `## Configuration Resolution` section before the Workflow section
3. Never hardcode output paths — no `.synapsync/` or absolute paths
4. Include a **post-production delivery step** at the end of the workflow

**Standard Configuration Resolution section** (copy this into new skills):

````markdown
## Configuration Resolution

`{output_dir}` is the directory where {skill-name} stores generated documents. Resolve it once at the start:

1. **User message context** — If the user's message contains file paths, extract `{output_dir}` from those paths
2. **Auto-discover** — Scan for `.agents/{skill-name}/` in `{cwd}`
3. **Ask the user** — If nothing found, ask where to save documents. Default suggestion: `.agents/{skill-name}/{project-name}/`

No AGENTS.md. No branded blocks. The output directory is resolved at runtime.
````

**Standard Post-Production Delivery step** (add at end of skill workflow):

```markdown
## Post-Production Delivery

After all documents are generated in `{output_dir}`, offer the user delivery options:

1. **Sync to Obsidian vault** — invoke the `obsidian` skill in SYNC mode
2. **Move to custom path** — user specifies a destination and files are moved there
3. **Keep in place** — leave files in `{output_dir}` for later use

Ask the user which option they prefer. If they choose option 1 or 2, move (not copy) the files to the destination.

**Obsidian invocation (option 1):**
- **Preferred**: `Skill("obsidian")`, then say "sync the files in {output_dir} to the vault"
- **Alternative**: Say "sync the output to obsidian" (triggers auto_invoke)
- **Subagent fallback**: Read the obsidian SKILL.md and follow SYNC mode workflow
```

## Commit Convention

This project uses conventional commits: `feat:`, `fix:`, `chore:`, `docs:`, etc. with optional scope in parentheses (e.g., `feat(agents): ...`).

## Validation Rules (for PRs)

- Cognitive name must be unique across the registry
- `manifest.json` must match the schema
- Content file referenced in `manifest.file` must exist
- Frontmatter in content file must match manifest metadata
- Category must be from the valid categories list
- Maximum 10 tags per cognitive
- Description maximum 100 characters
- Skills that produce output MUST use `{output_dir}` variable (never hardcoded paths)
- Skills that produce output MUST include a `## Configuration Resolution` section
- No `output_dir` field in SKILL.md frontmatter (resolved at runtime)
- Skills that produce output MUST include a post-production delivery step
- Modular skills (2+ modes) MUST have `## Asset Loading (Mode-Gated)` section
- SKILL.md frontmatter MUST NOT contain `changelog:` key (use manifest.json)
- SKILL.md body MUST NOT contain `## Version History` section

## Skill Update Protocol

Every time a cognitive is modified — any change, no matter how small — the following files MUST be updated. This is the single most common operation in this repo and the most error-prone. Follow this checklist exactly.

### Step 1: Make the Change

Edit the content file (`SKILL.md`, assets, etc.) with the actual change.

### Step 2: Version Bump (4 files)

All 4 version sources must stay in sync:

| File | Field | Format | Example |
|------|-------|--------|---------|
| `SKILL.md` frontmatter | `metadata.version` | `"X.Y"` (no patch) | `"3.8"` |
| `manifest.json` | `version` | `"X.Y.0"` (full semver) | `"3.8.0"` |
| `registry.json` | `version` (in the cognitive's entry) | `"X.Y.0"` | `"3.8.0"` |
| `README.md` | Version column in "Available Cognitives" table | `X.Y.0` | `3.8.0` |

**When to bump:**
- **Minor** (X.Y → X.Y+1): New features, new sections, behavior changes, expanded patterns — this is the default for most changes
- **Major** (X → X+1): Breaking changes, mode restructuring, consolidations
- **Patch** (X.Y.Z → X.Y.Z+1): NOT used — frontmatter has no patch slot, so every change bumps minor

### Step 3: Changelog (2 files)

| File | What to update |
|------|---------------|
| `manifest.json` | Prepend a new entry to the `changelog[]` array with `version`, `date` (today), and `changes[]` |
| `CHANGELOG.md` | Add entry under `[Unreleased]` in the appropriate subsection (Added, Changed, Fixed, Removed) |

### Step 4: Timestamp

| File | Field | Value |
|------|-------|-------|
| `manifest.json` | `updatedAt` | Today's date in ISO 8601 (`"YYYY-MM-DDT00:00:00Z"`) |

### Step 5: Conditional Updates

Check if any of these apply:

| Condition | Files to update |
|-----------|----------------|
| Description or tags changed | `manifest.json` (`description`, `tags`) AND `registry.json` (must match) |
| Cross-skill behavior changed | Other skills' `## Integration with Other Skills` tables |
| Assets added or removed | `manifest.json` `assets` counts |
| New provider added | `manifest.json` AND `registry.json` `providers[]` |
| Convention changed | `CLAUDE.md` (templates, validation rules) |
| Obsidian skill changed significantly | `README.md` "Obsidian Integration" section |

### Step 6: Post-Update Validation

Verify these constraints before committing:

- [ ] Frontmatter `"X.Y"` matches manifest `"X.Y.0"` (ignoring patch)
- [ ] `manifest.json` version = `registry.json` version
- [ ] `registry.json` version = `README.md` table version
- [ ] `description` ≤ 100 characters
- [ ] `tags` ≤ 10 items
- [ ] No `changelog:` key in SKILL.md frontmatter
- [ ] No `## Version History` section in SKILL.md body
- [ ] If modular (2+ modes): `## Asset Loading (Mode-Gated)` section exists
- [ ] If produces output: `## Configuration Resolution` section exists + `{output_dir}` used + post-production delivery exists
- [ ] If invokes obsidian: uses `Skill("obsidian")` or auto_invoke — obsidian skill handles all vault I/O

### Quick Reference: Files Touched per Skill Change

```
cognitives/skills/{category}/{name}/SKILL.md    ← content + frontmatter version
cognitives/skills/{category}/{name}/manifest.json ← version + changelog + updatedAt
registry.json                                     ← version
README.md                                         ← version in table
CHANGELOG.md                                      ← [Unreleased] entry
```

Minimum: **5 files** per change. With conditional updates, potentially more.

## Working on This Repo

- No build steps, no tests, no package manager — this is a pure content registry
- Changes typically involve adding new cognitives or updating existing ones
- **Always follow the Skill Update Protocol above** when modifying any cognitive
- Always keep `registry.json` in sync when adding/removing cognitives
- Use `make release` to bump the registry version, tag, and push
- Empty category directories are intentional placeholders
