# Brain Directory Configuration

Helper for persisting `{brain_dir}` to AGENTS.md. For the resolution algorithm (read → ask → validate), see [brain-resolve.md](brain-resolve.md).

---

## Resolution Algorithm

Run this **before** any path-dependent step in LOAD or SAVE:

### Step 1 — Read AGENTS.md

1. Read `{cwd}/AGENTS.md`
2. Scan for a `<!-- synapsync-skills:start -->` … `<!-- synapsync-skills:end -->` block
3. If the block exists → find `## Configuration` table → look for a `brain_dir` row
4. If the `brain_dir` row exists → extract the value, set `{brain_dir}`, done

### Step 2 — Ask the User (MANDATORY)

If AGENTS.md doesn't exist, has no SynapSync Skills block, or the block has no `brain_dir` key in the Configuration table, **ask the user**:

> **NEVER choose the default on behalf of the user.** You MUST call `AskUserQuestion` and wait for their selection. Proceeding without asking is a violation of the Configuration Resolution convention.

```
AskUserQuestion:
  question: "Where should brain documents be stored?"
  header: "Brain dir"
  options:
    - label: "Default (Recommended)"
      description: ".agents/staging/project-brain/"
```

The built-in "Other" option (shown as "Write your custom path") lets the user type a relative path directly. Set `{brain_dir}` to the chosen or typed path, then **persist** to AGENTS.md (see Persistence Rules below).

### Step 3 — Validate

After resolving `{brain_dir}`:
- If the directory does not exist yet, that's OK — SAVE INIT will create it
- If `{brain_dir}` points to a path that cannot be created (e.g., inside a read-only directory), report the error and ask for a different path

---

## Persistence Rules

After resolving `{brain_dir}`, persist the value to AGENTS.md so future sessions skip the resolution.

| # | Scenario | Action |
|---|----------|--------|
| 1 | No AGENTS.md exists | Create `{cwd}/AGENTS.md` with the full branded block (template below) including the `brain_dir` row |
| 2 | AGENTS.md exists, no `<!-- synapsync-skills:start -->` block | Append the full branded block at the end of the file (with a blank line for separation) |
| 3 | Block exists, no `## Configuration` section | Add `## Configuration` section with the `brain_dir` row before the closing `---` line |
| 4 | Block exists, `## Configuration` exists, no `brain_dir` row | Add `brain_dir` row to the Configuration table |
| 5 | Block exists, `## Configuration` exists, `brain_dir` value is different | Update the `brain_dir` row in place |
| 6 | Block exists, `## Configuration` exists, `brain_dir` value is the same | No-op |

### Block to Write (cases 1 and 2)

```markdown

<!-- synapsync-skills:start -->
# {Project Name} Skills Guidelines

## How to Use This Guide

This section of the `AGENTS.md` file contains project-specific guidelines and available skills from [SynapSync Registry](https://github.com/SynapSync/synapse-registry).

## Configuration

| Key | Value | Description |
|-----|-------|-------------|
| `brain_dir` | {brain_dir} | Session memory for AI agents — load context, save sessions, evolve knowledge |

---
<!-- synapsync-skills:end -->
```

When appending (case 2), add a blank line before the block for separation.

