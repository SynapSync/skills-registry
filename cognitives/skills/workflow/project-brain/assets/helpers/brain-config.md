# Brain Directory Configuration

Helper for persisting `{brain_dir}` configuration. For the resolution algorithm (discover → ask → validate), see [brain-resolve.md](brain-resolve.md).

---

## Resolution Algorithm

Run this **before** any path-dependent step in LOAD or SAVE:

### Step 1 — Auto-Discover

1. Check if the user's message contains file paths → extract `{brain_dir}` from those paths
2. Scan for `.agents/project-brain/` in `{cwd}`
3. Look for existing brain documents (files matching `*-brain.md` pattern)
4. If found → set `{brain_dir}`, done

### Step 2 — Ask the User (MANDATORY)

If no brain directory was discovered, **ask the user**:

> **NEVER choose the default on behalf of the user.** You MUST call `AskUserQuestion` and wait for their selection. Proceeding without asking is a violation of the Configuration Resolution convention.

```
AskUserQuestion:
  question: "Where should brain documents be stored?"
  header: "Brain dir"
  options:
    - label: "Default (Recommended)"
      description: ".agents/project-brain/{project-name}/"
```

The built-in "Other" option (shown as "Write your custom path") lets the user type a relative path directly. Set `{brain_dir}` to the chosen or typed path.

### Step 3 — Validate

After resolving `{brain_dir}`:
- If the directory does not exist yet, that's OK — SAVE INIT will create it
- If `{brain_dir}` points to a path that cannot be created (e.g., inside a read-only directory), report the error and ask for a different path

---

## Persistence

After resolving `{brain_dir}`, the path is stored in the brain documents themselves (README headers, document metadata). No external config file needed.

When saving brain documents, include the resolved `{brain_dir}` path in the document so future sessions can auto-discover it.
