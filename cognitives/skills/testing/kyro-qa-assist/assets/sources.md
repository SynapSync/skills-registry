# Sources — Where Project Knowledge Lives and How to Read It

## Allowed commands

This list is exhaustive. If a command is not here, do not run it.

**Shell glue (allowed)**

- `cd <repo>` or `git -C <repo>` to target the repository; prefer running from the repository root.
- Pipes into read-only text filters: `head`, `tail`, `sed -n`, `grep`, `wc`, `sort`, `uniq`, `cut`.
- `ls`, `test -e|-d|-f`, `cat` on files, `echo`, shell variables, and `2>/dev/null` / `2>&1`.
- Never redirect output into a file (`>`, `>>`, `tee`), never use `rm`, `mv`, `cp`, `mkdir`, `touch`, `chmod`, or an editor. Prefer the host's Read/Glob/Grep tools when it has them.

**git (read-only)**

- `git rev-parse`, `git status --porcelain` (to observe only), `git branch --list`, `git branch -r`
- `git tag --list`, `git tag --sort=-v:refname`, `git describe --tags`
- `git log` (any read flags: `--oneline`, `--no-merges`, `--stat`, `--name-status`, `--format`, `--follow`, `-S`, `-G`, `--grep`, `--since`, `--until`, `-- <path>`)
- `git diff <a>..<b>` / `git diff <a>...<b>` with read flags (`--stat`, `--name-status`, `--find-renames`, `-- <path>`)
- `git show <rev>`, `git show <rev>:<path>`
- `git ls-tree <rev> [<path>]`, `git cat-file -e|-p|-t <rev>:<path>` — check a file at a given version
- `git merge-base <a> <b>`, `git blame <path>`, `git ls-files`, `git grep [<rev>]`

**Never run builds or tests** (`npm run …`, `make`, `pytest`, …), even read-looking ones: they can write files, take long, and their result is not a manual check. Recommend them to the tester or a developer instead.

Never run `checkout`, `switch`, `reset`, `stash`, `commit`, `merge`, `rebase`, `fetch`, `pull`, `push`, `tag <name>`, `branch <name>`, `worktree`, or `clean`. To see an old version of a file, use `git show <rev>:<path>`, never a checkout.

**kyro (read-only)**

- `kyro status [brief|full|debt] [--kyro-scope <scope>] [--json]` — scope summary, task and ADR summary (`full`), debt by status (`debt`)
- `kyro scope list` and `kyro scope inspect <scope>`
- `kyro analyze [--matrix] [--kyro-scope <scope>] [--json]` — `--matrix` shows how much stored backing each spec scenario has
- `kyro trace [--kyro-scope <scope>] [--json] [--tail N] [--type <event>]` — never with `--clear`

Forbidden even though they sound like reads: `kyro context-pack` and `kyro doctor` (they write trace events), every `kyro debt …`, `kyro adr …`, `kyro scenario …`, `kyro scope set-active|complete|reopen|discard|retire`, and anything that writes evidence, reviews, plans, repairs, or closes.

**Files**: read anything with Read, Glob, and Grep. Never write.

## Detecting the tier

1. `git rev-parse --is-inside-work-tree` — not a repo: say so and stop investigating history; you can still read code.
2. `.agents/kyro/` exists on disk (`test -d .agents/kyro` or Glob; it is often gitignored, so `git ls-files` is not enough)? If not → **T3**.
3. `kyro status --json` succeeds and returns a parseable envelope? → **T1**. Fails, is missing, or returns something you cannot parse → **T2**. Do not try to install or update Kyro.

## Kyro artifact map (T1/T2)

All paths are relative to the repository root.

| What | Where | Why it matters to QA |
|------|-------|----------------------|
| Active scope state | `.agents/kyro/scopes/{scope}/sprint.json` | Objective, success criteria, `spec`, `debt[]`, `adrs[]`, active sprint |
| Requirements and scenarios | `sprint.json` → `spec.requirements[]` (R*), `spec.scenarios[]` (S*: given/when/then) | Ready-made acceptance scenarios and their requirement |
| Non-goals and open questions | `sprint.json` → `spec.nonGoals[]`, `spec.openQuestions[]` | What is intentionally out of scope; what is still undecided |
| Closed sprints | `.agents/kyro/scopes/{scope}/archive/sprint-NNN-{slug}.json` and `.md` | Per task: description, `acceptance_criteria`, `files_to_touch`, `evidence` (validations run, files changed), `verdict`; sprint close date in the archive front matter / record |
| Debt | `sprint.json` → `debt[]` (id, title, priority, status, note) or `kyro status debt` | Known weaknesses; open debt near a change raises risk |
| Decisions | `sprint.json` → `adrs[]` or `kyro status full` | Why something behaves the way it does |
| Learned rules | `.agents/kyro/scopes/rules.md` (if present) | Project conventions and past mistakes |
| Matured ideas | `.agents/kyro/plan/`, `.agents/kyro/analysis/`, `.agents/kyro/constitution/` | Original intent, before planning |
| Work briefs | `.agents/kyro/work/{id}/brief.md`, `work.json` | Intent and tasks of lighter-weight Work items |
| Project manifest | `.agents/kyro/project.json` | Artifact root and conventions |

Field names can vary between Kyro versions. If a field is missing, say so; do not guess its content.

## Joining git changes to Kyro intent

Kyro does not store commit hashes on tasks. Build the association from, in order of strength:

1. A task id (`T3.1`), scope name, or issue id in the commit message → `[Verified in code/git]` for the link.
2. The commit's files overlap a task's `files_to_touch` or its evidence "files changed", **and** the commit date falls before that sprint's close → `[Inferred]`, show both facts.
3. File overlap only → `[Inferred]`, weak; say so.
4. No overlap → the change has no documented intent; say so. This is important: undocumented changes deserve extra testing.

Never present the join as complete. Report changes you could not map.

**When only plans exist** (matured ideas or Work briefs, but no scope or sprint archive covers the range): use them as Claims and Purpose, say there is no task-level evidence or Kyro verdict, and match them to commits by topic, dates, and the files they name — `[Inferred]`.

## Intent sources outside Kyro (all tiers, essential in T3)

Read these before concluding "no documented intent": `CHANGELOG.md` / release notes, commit message bodies, `README.md` and `docs/`, migration guides, and design docs in the repository. They are claims made by the authors — label them `[Declared in plan/docs]`, naming the file. They are weaker than an approved plan: a commit body says what the author believes they did.

## Range defaults

- Release question with no range: latest tag → `HEAD`; if the tester names a version, find its tag (`git tag --list '*<version>*'`, tags often carry a `v` prefix).
- Release tags often sit on merge commits. Use `git log --no-merges <a>..<b>` for the content of the range, and do not rely on `git describe` across branches.
- Branch question: `git merge-base <main> <branch>` → `<branch>`, and diff with `...`.
- Ask which build or branch the tester actually has installed when it could differ from the range.
- Confirm the range before deep investigation. If the tester named two unambiguous refs, state the range and proceed.

## Large ranges

Start with `git diff --stat` / `--name-status` and group by top-level module. Explain modules in functional terms first. Read hunks only for the areas the tester cares about or that look risky. Always say which areas you did not read.

## Kyro Lens

If the team uses Kyro Lens (a read-only web UI for Kyro workspaces), you can suggest the tester open a scope or sprint there to see the evidence visually. It is optional; never depend on it.
