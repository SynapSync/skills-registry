# Changelog

All notable changes to the SynapSync Registry are documented in this file.

Format based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

---

## [Unreleased]

### Added
- **obsidian** skill (integrations, v3.8.0): RULE 0 "SKILL BEFORE MCP" guardrail — agents must follow SKILL.md workflow before calling any `mcp__obsidian__*` tools directly. Cross-Skill Invocation section with 3-priority invocation guide (Skill tool, natural language trigger, direct SKILL.md read). Expanded `auto_invoke` from 8 to 31 patterns covering EN/ES, SYNC/READ, and vault variants.

### Changed
- **project-brain** skill (workflow, v2.7.0): Strengthened mandatory user prompt for `brain_dir` resolution — agents can no longer silently choose the default path. New RULE 7 (NO SILENT DEFAULTS). `brain-config.md` Step 2 marked MANDATORY. `SAVE.md` Step 0 includes enforcement note.
- **code-analyzer** skill (analytics, v2.5.0): Post-Production Delivery updated with explicit obsidian skill invocation block and MCP guardrail.
- **universal-planner** skill (planning, v3.5.0): EXECUTE mode Post-Production Delivery updated with obsidian invocation block. Integration table updated with explicit invocation instructions and MCP guardrail.
- **sprint-forge** skill (workflow, v1.6.0): Integration table updated with explicit obsidian skill invocation instructions and MCP guardrail.
- **project-brain** skill (workflow, v2.6.0): Integration table updated with explicit obsidian skill invocation instructions and MCP guardrail.
- **CLAUDE.md**: Post-Production Delivery convention template updated with obsidian invocation block (`Skill("obsidian")`, natural language fallback, subagent fallback, MCP guardrail).

---

## [1.1.0] - 2026-02-20

### Added
- **Lazy Asset Loading convention** — modular skills (2+ modes) must include `## Asset Loading (Mode-Gated)` section with explicit "Do NOT Read" column to prevent agents from reading unnecessary assets.
- **Quick Start convention** — skills use imperative "Assets to read now:" instead of passive references.
- **universal-planner** skill (planning, v3.4.0): Asset Loading (Mode-Gated) section, Quick Start section, lightweight `output-resolve.md` helper (fast path for `output_dir` resolution). Trimmed `config-resolver.md`.
- **obsidian** skill (integrations, v3.7.0): Quick Start section with mode-specific asset references.
- **project-brain** skill (workflow, v2.5.0): Lightweight `brain-resolve.md` helper for LOAD mode (~170 tokens vs ~1,570). LOAD no longer reads `brain-config.md`.

### Changed
- **obsidian** skill (integrations, v3.6.0): Mode-gated lazy asset loading — agents only read assets for the active mode (SYNC or READ). Extracted Tool Dependencies to `assets/helpers/tool-reference.md`.
- **project-brain** skill (workflow, v2.4.0): Mode-gated lazy asset loading.
- **sprint-forge** skill (workflow, v1.5.0): Asset Loading (Mode-Gated) table added.
- **code-analyzer** skill (analytics, v2.4.0): Removed changelog from frontmatter, removed Version History section.
- Removed `changelog:` key from all SKILL.md frontmatters (canonical source: `manifest.json`).
- Removed `## Version History` sections from all SKILL.md bodies.

### Fixed
- `output_dir` never silently falls back to staging — always asks the user first.
- **project-brain**: Simplified `brain_dir` default path and config prompt.

---

## [1.0.2] - 2026-02-19

### Added
- **Branded AGENTS.md block** — `<!-- synapsync-skills:start/end -->` delimiters with shared `## Configuration` table. Any skill can add its config keys to the table. Single source of truth for skill configuration across sessions.
- **Configuration Resolution convention** — skills persist config keys (output paths, vault destinations) in AGENTS.md branded block. 6-case persistence rules for block creation/update.

### Changed
- **project-brain** skill (workflow, v2.3.0): Branded AGENTS.md block format with shared Configuration table (replaces hidden `<!-- synapsync:config -->` HTML comment). Updated `brain-config.md` helper with 6-case persistence rules.
- **obsidian** skill (integrations, v3.5.0): Branded AGENTS.md block support. New `vault_destination` config key — vault path remembered across sessions, no re-prompting.
- **sprint-forge** skill (workflow, v1.4.0): Branded AGENTS.md block support. New `output_dir` config key — supplements auto-discovery and re-entry prompts.
- **universal-planner** skill (planning, v3.3.0): Branded AGENTS.md block support. New `output_dir` config key — supplements deterministic staging fallback.
- **code-analyzer** skill (analytics, v2.3.0): Branded AGENTS.md block support. New `output_dir` config key — supplements deterministic staging fallback.

---

## [1.0.1] - 2026-02-19

### Added
- **project-brain** skill (workflow, v2.0.0): Session memory for AI agents with LOAD and SAVE modes. SAVE has INIT (create new) and UPDATE (incremental merge) sub-modes. Auto-discovery of brain documents. Standard brain document format with Project Identity, Active State, Session Log (newest-first), Accumulated Context (append-only), Next Steps, and Key Files. Backward-compatible parsing: v2.0, v1.0, and free-form. Modular assets: 2 modes, 1 helper, 1 template.
- **project-brain** skill (workflow, v2.1.0): Pre-write backup (`.bak`), session IDs for idempotent saves, semantic duplicate detection, session retrospective section, language separation (conversation in user's language, brain document always in English).

### Changed
- **obsidian** skill (integrations): Corrected MCP tool prefixes, params, and version regex.

### Fixed
- **sprint-forge** skill (workflow): Corrected `{output_dir}` discovery docs in INIT Step 2.

---

## [1.0.0] - 2026-02-17

### Added
- **sprint-forge** skill (workflow, v1.1.0): Adaptive sprint workflow with 3 modes (INIT, SPRINT, STATUS). Deep analysis, evolving roadmap, one-at-a-time sprint generation, formal debt tracking with inheritance, re-entry prompts for context persistence. Carry-over task state `[>]`, execution metadata, disposition actions, findings consolidation, evidence fields, "New Technical Debt Detected" retro subsection. Language-agnostic. Modular assets: 3 modes, 4 helpers, 4 templates.
- **Deterministic staging pattern** — `.agents/staging/{skill-name}/{project-name}/` convention for all output-producing skills.
- Post-production delivery step convention for output-producing skills.

### Changed
- **sprint-forge** skill (workflow, v1.3.0): Interactive path resolution replacing rigid staging pattern. Removed post-production delivery from INIT/SPRINT modes. Added Step 0 to SPRINT/STATUS for locating `{output_dir}`.
- **universal-planner** skill (planning, v3.2.0): Migrated to staging pattern, renamed `{output_base}` to `{output_dir}`, added post-production delivery to EXECUTE mode.
- **code-analyzer** skill (analytics, v2.2.0): Migrated to staging pattern, renamed `{output_base}` to `{output_dir}`, added post-production delivery.
- **obsidian** skill (integrations, v3.4.0): Migrated to staging convention, batch-move-from-staging workflow in SYNC mode.

---

## [0.9.0] - 2026-02-14

### Added
- **universal-planner** skill (planning, v3.1.1): Comprehensive test suite — 6 test cases covering all planning sub-modes (NEW_FEATURE, REFACTOR, BUG_FIX with fast path, TECH_DEBT, NEW_PROJECT, ARCHITECTURE).
- **obsidian** skill (integrations, v3.3.0): Filesystem fallback for SYNC mode — MCP changed from hard dependency to optional. Added Step 0 access mode detection, YAML serialization guidance for filesystem writes.

### Fixed
- **universal-planner** skill (planning, v3.1.0): Audit remediation — 25 findings resolved. Unified task granularity (Sprint/Phase/Task/Subtask), PROGRESS.md template, before/after code fields, BUG_FIX fast path, sprint failed status, plan freshness check.
- **obsidian** skill (integrations): Prevented vault path defaulting to `agents/` folder.

---

## [0.8.0] - 2026-02-13

### Added
- **obsidian** skill (integrations, v3.2.0): Full MCP tool coherence — all 13 Obsidian MCP tools documented and integrated (was 7). Added `delete_note`, `patch_note`, `update_frontmatter`, `get_frontmatter`, `get_notes_info`, `move_note`. Optimized cross-ref-validator with `update_frontmatter`, priority-ranking with `get_notes_info` metadata fast path. Added archive/delete and move/reorganize workflows.
- **universal-planner** skill (planning, v3.0.0): Consolidated universal-planner + universal-planner-executor into unified skill with PLAN and EXECUTE modes. PLAN mode produces documentation only; EXECUTE mode implements sprints with dual roles (Senior Developer + Scrum Master). Added PLAN.md and EXECUTE.md mode files, RETRO template, decision-log, code-quality-standards.

### Fixed
- **obsidian** skill (integrations, v3.1.0): Audit remediation — 20 findings resolved across 3 rounds. Added i18n documentation, tool parameter contracts, reconciled type taxonomy to 14 types, ambiguous intent disambiguation, batch size limits, negative weights for superseded/archived.

---

## [0.7.0] - 2026-02-12

### Added
- **obsidian** skill (integrations, v3.0.0): Consolidated obsidian-sync + obsidian-reader into unified skill with SYNC and READ modes. Progressive disclosure pattern. 4 shared helpers (frontmatter-generator, cross-ref-validator, batch-sync-pattern, priority-ranking).
- **Modular assets pattern** — skills extract modes, helpers, templates, and validators to `assets/` directory for progressive disclosure and reduced SKILL.md size.
- Obsidian markdown standard specification (v1.0) absorbed as internal asset of obsidian skill.
- Obsidian linter validator extracted and shared across skills.

### Changed
- **universal-planner** skill (planning, v2.1.0): Refactored to modular assets pattern — extracted 6 modes, config-resolver, 5 templates, output validator. Reduced SKILL.md to 591 LOC.
- **code-analyzer** skill (analytics, v2.1.0): Refactored to assets pattern — extracted 2 templates and diagram guidelines. Reduced SKILL.md from 524 to 343 LOC.
- **obsidian-sync** (pre-consolidation): Extracted helpers to `assets/`, reduced SKILL.md to 293 LOC.
- Migrated default `output_base` from `~/obsidian-vault/` to `~/.agents/`.

### Removed
- **project-planner** skill — superseded by universal-planner.
- **sdlc-planner** skill — superseded by universal-planner.
- **universal-planner-executor** skill — merged into universal-planner as EXECUTE mode.
- **obsidian-sync** skill — merged into unified obsidian skill.
- **obsidian-reader** skill — merged into unified obsidian skill.
- **obsidian-md-standard** skill — absorbed into obsidian skill as `assets/standards/`.
- Empty placeholder directories removed.

### Fixed
- Comprehensive certification fixes across all cognitives.
- Removed all broken external references for self-contained skills.

---

## [0.6.0] - 2026-02-11

### Added
- **obsidian-sync** skill (integrations, v2.0.0): Obsidian vault sync with MCP integration, frontmatter generation, cross-reference validation.
- **obsidian-reader** skill (integrations, v1.0.0): Obsidian vault reader with search, ranking, and compliance checking.
- **obsidian-md-standard** skill (integrations, v1.0.0): Obsidian markdown standard specification with universal frontmatter schema, type taxonomy, and wiki-link conventions.

### Changed
- **code-analyzer** skill (analytics, v2.0.0): Migrated to obsidian-md-standard output format, added v1/v2/v3 analysis depth levels, per-project output configuration via `cognitive.config.json`.

---

## [0.5.0] - 2026-02-04

### Added
- **universal-planner** skill (planning, v2.0.0): Unified planning skill with 6 sub-modes (NEW_PROJECT, NEW_FEATURE, REFACTOR, BUG_FIX, TECH_DEBT, ARCHITECTURE). Obsidian-native markdown output.
- **universal-planner-executor** skill (planning, v1.0.0): Sprint execution with dual roles (Senior Developer + Scrum Master), phase-by-phase execution, verification gates, decision logging.
- Optional sprint generation with user prompt.

---

## [0.4.0] - 2026-01-31

### Added
- **sdlc-planner** skill: Initial SDLC planning skill (later superseded by universal-planner).

---

## [0.3.0] - 2026-01-29

### Added
- **code-analyzer** skill (analytics, v1.0.0): Code module analysis with structured technical reports, Mermaid diagrams, execution flow tracing, dependency mapping.

### Changed
- **project-planner** skill: Added context-first planning rules, refactored to planning-only skill (no execution).

---

## [0.2.0] - 2026-01-28

### Added
- **project-planner** skill (workflow, v1.0.0): Project planning framework with analysis, planning, and execution phases.
- **cognitive-register** tool (general, v1.0.0): Internal tool for automating cognitive registration into the registry.
- **feature-branch-manager** agent (general, v1.0.0): Git workflow agent for feature branch creation, pushing changes, and PR creation. Claude-only.
- `CLAUDE.md` project documentation with conventions, validation rules, and naming standards.
- Expanded provider lists across all cognitives.

---

## [0.1.0] - 2026-01-28

### Added
- Initial repository structure with `registry.json` central index and `cognitives/` directory.
- **skill-creator** skill (general, v3.0.0): Meta-skill for creating new skills with basic and advanced templates.
- Cognitive type definitions: skill, agent, prompt, workflow, tool.
- Category taxonomy: general, frontend, backend, database, devops, security, testing, analytics, automation, integrations, planning, workflow.
- Provider support: claude, openai, cursor, windsurf, copilot, gemini.
- `manifest.json` schema and naming conventions.
- Comprehensive documentation (`README.md`, `CONTRIBUTING.md`).
