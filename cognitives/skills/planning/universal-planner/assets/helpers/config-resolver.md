# Configuration Resolver

## Purpose

Resolve the `{output_dir}` path that determines where all planning output documents are stored. For quick resolution, use [output-resolve.md](output-resolve.md) first — it covers the common case. This file contains the full resolution rules and error handling for first-time setup.

## Resolution Algorithm

1. **User message context** — If the user's message contains file paths, extract `{output_dir}` from those paths
2. **Auto-discover** — Scan for `.agents/universal-planner/` in `{cwd}`
3. **Ask the user** (first-time only) — offer default or custom path

**IMPORTANT**: Skills must NEVER silently write to a path without asking the user first. The default path is only an **option** offered to the user, not an automatic fallback.

## Default Path (offered as option)

When asking the user, offer this as the default option:

```
.agents/universal-planner/{project-name}/
```

- `{project-name}` — inferred from the current directory name or git repository name

## Workflow

Follow these steps at the **beginning** of any skill execution that generates output:

### Step 1: Auto-Discover

1. Check if the user's message contains file paths → extract `{output_dir}`
2. Scan for `.agents/universal-planner/` in `{cwd}`
3. If found → set `{output_dir}`, **skip to Step 3** (present to user)
4. If not found → continue to Step 2 (ask user)

### Step 2: Ask the User

If no `output_dir` is discovered, **ask the user** where they want output stored:

1. Infer the project name from git repo or directory name
2. Present two options:

```
AskUserQuestion:
  question: "Where should output documents be stored?"
  header: "Output dir"
  options:
    - label: "Default (Recommended)"
      description: ".agents/universal-planner/{project-name}/"
    - label: "Custom path"
      description: "Provide a relative path from project root"
```

3. If user chooses **default** → set `{output_dir}` = `.agents/universal-planner/{project-name}/`
4. If user chooses **custom** → set `{output_dir}` to the path the user provides

### Step 3: Present to User

Inform the user of the resolved path:

```
Output documents will be stored in: {output_dir}/

Proceed?
```

### Step 4: Create Directory and Use {output_dir}

1. Create the directory if it doesn't exist
2. Throughout the skill, use `{output_dir}` as a variable:

```
{output_dir}/planning/{feature-name}/
{output_dir}/technical/{module-name}/
{output_dir}/docs/{category}/
```

The skill replaces `{output_dir}` with the resolved value.

## Path Details

**Rules:**

- **Path type:** Relative to project root. Can be any user-configured value (e.g., `docs/planning`, `output/reports`, or `.agents/universal-planner/...`).
- **No trailing slash in the stored value:** `docs/planning` (not `docs/planning/`). When using the value, append `/` as needed (e.g., `{output_dir}/planning/`).
- **Resolved once per session:** After first resolution, the value is reused for the entire session.

## Error Handling

### Directory Creation Fails

If the directory cannot be created (permissions issue):

```
ERROR: Cannot create directory: {output_dir}

Please check permissions or specify an alternative path.
```

### Project Name Cannot Be Inferred

If neither git remote nor directory name yields a usable slug:

```
Could not infer the project name. What should I call this project?
(This will be used for the default output path: .agents/universal-planner/{your-name}/)
```
