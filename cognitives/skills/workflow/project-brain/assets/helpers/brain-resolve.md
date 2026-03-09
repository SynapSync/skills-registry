# Brain Directory Resolution

Resolves `{brain_dir}` — the directory where brain documents are stored. Run this before any path-dependent operation.

## Algorithm

1. **User message context** — If the user's message contains file paths to brain documents, extract `{brain_dir}` from those paths
2. **Auto-discover** — Scan for `.agents/project-brain/` in `{cwd}`, or look for existing brain documents (files matching `*-brain.md` pattern)
3. **Ask the user** — If nothing found, ask via `AskUserQuestion`:
   - Default (Recommended): `.agents/project-brain/{project-name}/`
   - Or custom path via "Other"
4. **Validate**: directory not existing is OK for SAVE INIT (will create). For LOAD, report error and ask for alternative path.

After resolution, `{brain_dir}` is set for all subsequent operations.
