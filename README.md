# SynapSync Registry

The official public registry of cognitives (skills, agents, prompts, workflows, tools) for AI-powered development.

## What is This?

SynapSync Registry is a collection of reusable AI instructions ("cognitives") that help AI assistants understand project patterns and conventions. Cognitives work across multiple providers (Claude, OpenAI, Cursor, Windsurf, Copilot, Gemini).

## Installing Cognitives

Use the `skills` CLI to install cognitives from any GitHub repository:

```bash
# Install cognitives from a registry repo
npx skills add owner/repo

# Update installed cognitives
npx skills update
```

## Registry Structure

```
synapse-registry/
├── README.md                           # This file
├── CLAUDE.md                           # Project conventions for AI agents
├── CHANGELOG.md                        # Release history
├── CONTRIBUTING.md                     # Guide for contributors
├── Makefile                            # Release automation (make release)
├── registry.json                       # Central index of all cognitives
│
├── cognitives/                         # Public registry content (downloadable)
│   ├── skills/                         # AI instruction skills
│   │   ├── general/                    # General-purpose skills
│   │   ├── analytics/                  # Data analysis
│   │   ├── planning/                   # Project planning & SDLC
│   │   ├── integrations/              # External services & Obsidian
│   │   └── workflow/                  # Sprint workflows & iterative processes
│   │
│   ├── agents/                         # Autonomous AI agents
│   │   └── general/
│   │
│   ├── prompts/                        # Reusable prompt templates
│   ├── workflows/                      # Multi-step AI workflows
│   └── tools/                          # External integrations
│
└── core/                               # Internal project tooling (not published)
    └── register/                       # Cognitive registration automation
```

## Available Cognitives

### Skills

| Name | Category | Version | Description |
|------|----------|---------|-------------|
| [skill-creator](cognitives/skills/general/skill-creator/) | general | 3.3.0 | Creates AI skills following SynapSync spec with templates and best practices |
| [universal-planner](cognitives/skills/planning/universal-planner/) | planning | 3.5.0 | Unified planning and execution skill for any software scenario with PLAN and EXECUTE modes |
| [code-analyzer](cognitives/skills/analytics/code-analyzer/) | analytics | 2.5.0 | Analyzes code modules and generates structured technical reports with architecture diagrams |
| [obsidian](cognitives/skills/integrations/obsidian/) | integrations | 3.9.0 | Unified Obsidian vault manager: sync documents, read notes, and search knowledge |
| [sprint-forge](cognitives/skills/workflow/sprint-forge/) | workflow | 1.8.0 | Adaptive sprint workflow — analysis, roadmap, iterative sprints, debt tracking, and context persistence |
| [project-brain](cognitives/skills/workflow/project-brain/) | workflow | 2.7.0 | Session memory for AI agents — load context, save sessions, evolve knowledge |

### Agents

| Name | Category | Version | Description |
|------|----------|---------|-------------|
| [feature-branch-manager](cognitives/agents/general/feature-branch-manager/) | general | 1.0.0 | Git workflow manager for feature branch creation, pushing changes, and PR creation |

---

## Obsidian Integration

The registry includes a unified **obsidian** skill that provides a bidirectional knowledge bridge between the agent's workspace and an [Obsidian](https://obsidian.md) vault. It turns Obsidian into a persistent knowledge layer for AI agents -- plans, reports, decisions, and documentation flow in and out of the vault seamlessly.

### Two Modes, One Skill

The obsidian skill operates in two modes, detected automatically from the user's intent:

| | SYNC Mode | READ Mode |
|---|---|---|
| **Direction** | Workspace **-->** Vault | Vault **-->** Agent |
| **Purpose** | Persist reports, plans, and docs into the vault | Retrieve knowledge from the vault to inform decisions |
| **Trigger** | "sync to obsidian", "save plan to obsidian" | "read from obsidian", "what do my notes say about X?" |
| **MCP Tools** | `write_note`, `patch_note`, `delete_note`, `move_note`, `update_frontmatter`, `list_directory` | `read_note`, `read_multiple_notes`, `search_notes`, `get_frontmatter`, `get_notes_info`, `get_vault_stats`, `manage_tags`, `list_directory` |
| **Fallback** | Filesystem write via `Write`/`Glob` if MCP is unavailable | Filesystem read via `Read`/`Glob`/`Grep` if MCP is unavailable |

### Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Agent Workspace                          │
│                                                                 │
│  universal-planner ──┐                                          │
│  code-analyzer ──────┤  produce .md files   ┌────────────────┐  │
│  (any skill)  ───────┘  ({output_dir}/*)   │    obsidian    │  │
│                                             │  SYNC mode     │  │
│                                             └───────┬────────┘  │
│                                                     │           │
│                                            Obsidian MCP Server  │
│                                                     │           │
│                                             ┌───────▼────────┐  │
│  Agent needs context ─────────────────────> │    obsidian    │  │
│  "what do my notes say about X?"            │  READ mode     │  │
│  "project status from obsidian"             └───────┬────────┘  │
│                                                     │           │
│                    structured context, summaries,    │           │
│                    source citations, priority ranking │           │
│                                                     ▼           │
│                              Agent uses knowledge to reason     │
└─────────────────────────────────────────────────────────────────┘
                                │
                                │  mcp__obsidian__*
                                ▼
                   ┌─────────────────────────┐
                   │     Obsidian Vault       │
                   │                         │
                   │  work/                  │
                   │    project-a/           │
                   │      plans/             │
                   │    project-b/           │
                   │      reports/           │
                   └─────────────────────────┘
```

### Prerequisites

The obsidian skill requires the **Obsidian MCP server** for full functionality:

1. **Install Obsidian** and open your vault
2. **Install the Local REST API plugin** in Obsidian (Community Plugins > Local REST API)
3. **Configure the MCP server** in your Claude Code settings (`~/.claude/.mcp.json`):

```json
{
  "mcpServers": {
    "obsidian": {
      "command": "npx",
      "args": ["-y", "obsidian-mcp"],
      "env": {
        "OBSIDIAN_API_KEY": "your-api-key",
        "OBSIDIAN_API_URL": "https://127.0.0.1:27124"
      }
    }
  }
}
```

### Usage Examples

```bash
# Write to Obsidian (SYNC mode)
"sync my planning docs to obsidian"
"create a technical report and save it to obsidian"

# Read from Obsidian (READ mode)
"what do my obsidian notes say about the project?"
"search obsidian for architecture decisions"

# Combined workflows
"analyze this module, generate a report, and save it to obsidian"
```

---

## Cognitive Types

| Type | Description | File |
|------|-------------|------|
| **Skill** | Reusable instructions for AI assistants | `SKILL.md` |
| **Agent** | Autonomous AI entities with specific behaviors | `AGENT.md` |
| **Prompt** | Reusable prompt templates with variables | `PROMPT.md` |
| **Workflow** | Multi-step processes combining agents and prompts | `WORKFLOW.yaml` |
| **Tool** | External integrations and functions | `TOOL.md` |

## Categories

| Category | Description |
|----------|-------------|
| `general` | General-purpose cognitives |
| `frontend` | UI, React, CSS, components |
| `backend` | APIs, servers, backend services |
| `database` | Database queries, migrations, ORMs |
| `devops` | CI/CD, infrastructure, deployment |
| `security` | Security analysis, vulnerability scanning |
| `testing` | Testing strategies, QA automation |
| `analytics` | Data analysis, research, benchmarking |
| `automation` | Task automation, workflows |
| `integrations` | External services, Obsidian, and knowledge management |
| `planning` | Project planning, SDLC, requirements, architecture |
| `workflow` | Sprint workflows, iterative processes, adaptive execution |

## Output Path Convention

Skills that produce output documents (reports, plans, analysis) resolve their output path from the **AGENTS.md branded block**. When no config exists (first-time use), skills **ask the user** for their preferred path before writing anything.

### How It Works

1. **Read** `AGENTS.md` → look for `output_dir` in the `<!-- synapsync-skills:start -->` Configuration table
2. If found → use it, done
3. If not found → **ask the user**: use default (`.agents/staging/{skill-name}/{project-name}/`) or provide a custom path
4. **Persist** the chosen value to AGENTS.md so future sessions skip the prompt

After generating output, skills offer **post-production delivery**: sync to Obsidian vault (via the `obsidian` skill -- never calling MCP tools directly), move to a custom path, or keep in place.

### For Skill Authors

If your skill produces output files, you **must**:
1. Use `{output_dir}` for all output paths (e.g., `{output_dir}/planning/`, `{output_dir}/technical/`)
2. Include a `## Configuration Resolution` section before the Workflow section (see [CLAUDE.md](CLAUDE.md) for the standard template)
3. Never hardcode output paths -- no `.synapsync/` or absolute paths
4. Never put `output_dir` in SKILL.md frontmatter
5. Never silently fall back to staging -- always ask the user when no config exists
6. Include a post-production delivery step at the end of the workflow

## Contributing

We welcome contributions! See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on how to submit your own cognitives.

### Quick Start

1. Fork this repository
2. Create a folder: `cognitives/{type}s/{category}/{your-cognitive-name}/`
3. Add `manifest.json` and your cognitive file (e.g., `SKILL.md`)
4. Add an entry to `registry.json`
5. Submit a Pull Request

## Links

- **Skills CLI**: `npx skills add owner/repo`
- **Issues**: https://github.com/SynapSync/skills-registry/issues

## License

All cognitives in this registry are individually licensed. Check each cognitive's `manifest.json` for its specific license.

The registry structure and tooling are licensed under MIT.
