# Output Directory Resolution

Resolves `{output_dir}` — the directory where planning documents are stored. Run this before any path-dependent operation.

## Algorithm

1. **User message context** — If the user's message contains file paths, extract `{output_dir}` from those paths
2. **Auto-discover** — Scan for `.agents/universal-planner/` in `{cwd}`
3. **Ask the user** — If nothing found, ask via `AskUserQuestion`:
   - Default (Recommended): `.agents/universal-planner/{project-name}/`
   - Or custom path via "Other"

After resolution, `{output_dir}` is set for all subsequent operations. See [config-resolver.md](config-resolver.md) for the full algorithm.
