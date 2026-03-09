# Skill Assets Pattern

**Version:** 1.0
**Created:** 2026-02-12
**Status:** Active

---

## Purpose

Define a standardized pattern for organizing supporting files (modes, templates, helpers, validators) within skills to:

1. **Reduce SKILL.md size** — Keep the main skill file concise (overview) while providing depth on-demand
2. **Improve maintainability** — Change templates/modes without touching SKILL.md
3. **Ensure autocontained skills** — All assets download with `npx skills add {skill-name}`
4. **Enable reusability** — Helpers and validators can be referenced across the skill

---

## When to Use Assets

### Use `assets/` when:

| Condition | Example |
|-----------|---------|
| **Template > 50 LOC** | Document templates with extensive frontmatter and structure |
| **Multiple modes** | Skill has 3+ modes with different workflows |
| **Complex helper (> 30 LOC)** | Configuration resolution, validation workflows |
| **Example configs needed** | Sample inputs, test cases, reference implementations |
| **JSON schemas** | Output validators, contracts for downstream skills |

### Keep inline when:

| Condition | Example |
|-----------|---------|
| **Template < 30 LOC** | Simple frontmatter-only templates |
| **Single mode** | No mode variations |
| **Simple workflow** | Linear, < 10 steps |
| **No examples** | Self-explanatory skill |

---

## Standard Structure

```
{skill-name}/
  ├── SKILL.md                # Main skill file (concise, 400-600 LOC)
  ├── manifest.json           # Skill metadata
  └── assets/                 # Supporting files (self-contained)
      ├── README.md           # What's in assets/ and why
      ├── modes/              # Mode-specific rules (if applicable)
      │   ├── MODE_A.md
      │   ├── MODE_B.md
      │   └── MODE_C.md
      ├── helpers/            # Reusable workflows
      │   ├── config-resolver.md
      │   └── error-handler.md
      ├── templates/          # Document templates
      │   ├── OUTPUT_A.md
      │   └── OUTPUT_B.md
      ├── validators/         # JSON schemas
      │   ├── output-schema.json
      │   └── README.md
      └── examples/           # Test cases, sample configs
          ├── test-case-1.md
          └── sample-config.json
```

---

## Directory Purposes

### `assets/README.md` (Required)

Purpose of the assets directory and what each subdirectory contains.

**Template:**
```markdown
# {Skill Name} Assets

Supporting files for {skill-name} following the assets pattern.

## Directory Structure
[Tree view]

## Purpose of Each Directory
### modes/
[What goes here]

### helpers/
[What goes here]

...
```

### `modes/` (Optional)

**When to use:** Skill has multiple modes with different workflows (e.g., NEW_PROJECT vs NEW_FEATURE)

**What goes here:** One file per mode with:
- When to use this mode
- Output structure for this mode
- Frontmatter additions (mode-specific fields)
- Workflow adjustments
- Examples

**Naming:** `MODE_NAME.md` (e.g., `NEW_PROJECT.md`, `REFACTOR.md`)

### `helpers/` (Optional)

**When to use:** Skill has reusable workflows that appear in multiple places

**What goes here:** Workflow documentation that:
- Can be referenced from SKILL.md
- Reduces duplication
- Provides step-by-step procedures

**Examples:**
- `config-resolver.md` — How to resolve `{output_dir}`
- `error-handler.md` — Standard error handling workflow
- `validation.md` — Input validation procedures

### `templates/` (Optional)

**When to use:** Skill generates documents with complex structure (> 50 LOC)

**What goes here:** Document templates with:
- Frontmatter (with `{{placeholders}}`)
- Section structure
- Inline examples
- Variable substitution markers

**Naming:** `OUTPUT_NAME.md` (e.g., `ANALYSIS.md`, `SPRINT.md`)

**Placeholder format:** Use `{{variable_name}}` for substitution

### `validators/` (Optional)

**When to use:** Skill produces output that downstream skills consume

**What goes here:**
- JSON schemas defining output contracts
- README explaining how to validate

**Example:**
- `output-schema.json` — JSON Schema for skill output
- `README.md` — Validation usage guide

### `examples/` (Optional)

**When to use:** Skill has complex inputs/outputs or needs test cases

**What goes here:**
- Test cases
- Sample configurations
- Reference implementations
- Smoke tests

---

## Referencing Assets in SKILL.md

### Pattern: Summary + Link

Instead of embedding full content inline, use:

```markdown
## Section Title

[Brief 2-3 sentence summary]

**Full details:** See [assets/path/to/file.md](assets/path/to/file.md)
```

### Example: Modes

**Before (inline):**
```markdown
## NEW_PROJECT Mode

[200 lines of mode documentation...]

## NEW_FEATURE Mode

[150 lines of mode documentation...]
```

**After (with assets):**
```markdown
## Planning Modes

This skill supports 6 planning modes:

- **NEW_PROJECT**: Full SDLC planning for greenfield → [details](assets/modes/NEW_PROJECT.md)
- **NEW_FEATURE**: Codebase-aware feature planning → [details](assets/modes/NEW_FEATURE.md)
- **REFACTOR**: Technical improvement planning → [details](assets/modes/REFACTOR.md)
- **BUG_FIX**: Bug investigation & fix planning → [details](assets/modes/BUG_FIX.md)
- **TECH_DEBT**: Technical debt reduction → [details](assets/modes/TECH_DEBT.md)
- **ARCHITECTURE**: Architecture changes → [details](assets/modes/ARCHITECTURE.md)

See individual mode files for detailed workflows and requirements.
```

### Example: Configuration Resolution

**Before (inline):**
```markdown
## Configuration Resolution

[80 lines of config resolution workflow...]
```

**After (with assets):**
```markdown
## Configuration Resolution

See [assets/helpers/config-resolver.md](assets/helpers/config-resolver.md) for the full workflow.

**Quick summary:**
1. Check user message for file paths → extract `{output_dir}`
2. Auto-discover → scan for `.agents/{skill-name}/` in `{cwd}`
3. If not found → **ask the user** (default `.agents/{skill-name}/{project-name}/` or custom path)
4. Use `{output_dir}/` for all output paths
```

---

## Benefits

### For Skill Authors

1. **Easier to maintain** — Update templates without touching SKILL.md
2. **Clearer structure** — Separation of overview vs depth
3. **Reusable components** — Helpers can be referenced multiple times
4. **Testable** — Validators and examples in assets/

### For Skill Users

1. **Faster onboarding** — SKILL.md is concise overview
2. **Depth on-demand** — Drill into details as needed
3. **Predictable structure** — All skills follow same pattern
4. **Self-contained** — No external dependencies

### For the Ecosystem

1. **Consistency** — All skills organized the same way
2. **Discoverability** — Assets are easy to find
3. **Composability** — Helpers can inform new skills
4. **Scalability** — Pattern works for 10 or 100 skills

---

## Migration Guide

### Step 1: Create Assets Structure

```bash
cd cognitives/skills/{category}/{skill-name}/
mkdir -p assets/{modes,helpers,templates,validators,examples}
```

### Step 2: Identify What to Extract

Read SKILL.md and identify:

| Content Type | Extract to |
|--------------|------------|
| Mode-specific workflows (> 50 LOC each) | `assets/modes/{MODE}.md` |
| Configuration/setup workflows (> 30 LOC) | `assets/helpers/{helper}.md` |
| Document templates (> 50 LOC) | `assets/templates/{OUTPUT}.md` |
| JSON schemas | `assets/validators/{schema}.json` |
| Test cases, examples | `assets/examples/{example}.md` |

### Step 3: Extract Content

For each identified section:

1. Create the file in appropriate `assets/` subdirectory
2. Copy content from SKILL.md
3. Add any mode-specific context
4. Ensure placeholders are marked with `{{variable}}`

### Step 4: Update SKILL.md

Replace extracted sections with:
- Brief summary (2-3 sentences)
- Link to assets file: `[details](assets/path/to/file.md)`

### Step 5: Create assets/README.md

Document what's in assets and why.

### Step 6: Validate

1. Check all references resolve
2. Verify no broken links
3. Test that skill still functions
4. Measure LOC reduction

**Target:** SKILL.md should be 400-600 LOC after extraction

---

## Examples

### Example 1: universal-planner

**Before:**
- SKILL.md: 1,012 LOC
- No assets/

**After:**
- SKILL.md: ~700 LOC (-30%)
- assets/modes/: 6 files (mode docs)
- assets/helpers/: 1 file (config resolver)
- assets/templates/: 5 files (document templates)
- assets/validators/: 2 files (JSON schema + README)
- assets/examples/: 1 file (smoke test)

**Impact:** Clearer overview, modes easily discoverable, templates maintainable

### Example 2: code-analyzer (hypothetical)

**Before:**
- SKILL.md: 525 LOC
- Inline templates for REPORT.md and REFACTOR.md

**After:**
- SKILL.md: ~350 LOC
- assets/templates/REPORT.md
- assets/templates/REFACTOR.md
- assets/validators/report-schema.json

---

## Anti-Patterns

### ❌ Don't Do This

1. **Creating assets for tiny content**
   - If template is 20 LOC, keep it inline

2. **Deep nesting**
   - `assets/modes/NEW_PROJECT/workflows/step1.md` ❌
   - `assets/modes/NEW_PROJECT.md` ✅

3. **Duplicating content**
   - Don't copy the same helper to multiple skills
   - Reference the same pattern or extract to shared docs

4. **External dependencies**
   - Assets should be self-contained
   - Don't reference files outside the skill directory

5. **Mixing concerns**
   - `assets/templates/` should only have templates
   - Don't put helpers in templates/

---

## Versioning

When a skill uses assets:

1. **manifest.json** should include:
   ```json
   {
     "assets": {
       "modes": 6,
       "helpers": 1,
       "templates": 5,
       "validators": 1
     }
   }
   ```

2. **Changelog** should track asset changes:
   ```json
   {
     "version": "2.1.0",
     "changes": [
       "Refactored to use assets pattern",
       "Extracted 6 modes to assets/modes/",
       "Reduced SKILL.md from 1012 to 700 LOC"
     ]
   }
   ```

---

## FAQ

### Q: Do all skills need assets/?

**A:** No. Only use assets if you have content that meets the criteria (templates > 50 LOC, multiple modes, complex helpers). Simple skills can remain inline.

### Q: Can I reference assets from another skill?

**A:** No. Each skill should be self-contained. If two skills need the same helper, document the pattern in `docs/standards/` and let each skill implement it.

### Q: What if my template is 45 LOC?

**A:** Use your judgment. The 50 LOC guideline is not strict. If extracting it improves clarity, do it.

### Q: Should I commit assets to git?

**A:** Yes. Assets are part of the skill and should be version controlled.

### Q: Can assets reference each other?

**A:** Yes, within the same skill. `assets/modes/NEW_PROJECT.md` can reference `assets/templates/ANALYSIS.md`.

---

## Related Standards

- **Obsidian markdown standard** — Output format for markdown documents (embedded in the obsidian skill at `assets/standards/obsidian-md-standard.md`)
- **manifest.json schema** — Skill metadata structure
- **SKILL.md frontmatter** — Skill configuration format

---

## Version History

- **1.0** (2026-02-12): Initial assets pattern specification based on universal-planner refactoring
