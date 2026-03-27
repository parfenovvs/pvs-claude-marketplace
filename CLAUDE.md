# pvs-claude-marketplace

Claude Code plugin marketplace — personal collection of skills distributed via the Claude Code plugin system.

## Purpose

This repository is a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces). It hosts skills that users install with `/plugin install <name>@pvs-claude-marketplace`. Each skill is a prompt-driven behaviour that extends what Claude can do in a project.

## Repository Structure

```
pvs-claude-marketplace/
├── README.md                        # Marketplace overview and install instructions
├── CLAUDE.md                        # This file — project structure and guidance for AI agents
├── .claude-plugin/
│   └── marketplace.json             # Marketplace catalog (required by Claude Code)
└── plugins/                         # One directory per plugin
    ├── commit/                      # Conventional commit message authoring
    │   └── skills/commit/SKILL.md
    ├── jj/                          # Jujutsu (jj) version control operations
    │   └── skills/jj/SKILL.md
    ├── new-feature/                 # New feature scaffolding and planning
    │   └── skills/new-feature/SKILL.md
    ├── pr/                          # Pull request creation
    │   └── skills/pr/SKILL.md
    └── tbd/                         # Trunk-based development planning
        └── skills/tbd/SKILL.md
```

## Marketplace Catalog

`.claude-plugin/marketplace.json` is the entry point for the Claude Code plugin system. It lists every plugin in this repo, their sources, categories, and tags. Claude Code reads this file when users run `/plugin marketplace add` or `/plugin install`.

Each plugin is self-contained under `plugins/<name>/`. The `"source"` field in each marketplace entry points to the plugin directory, and Claude Code auto-discovers skills inside `<plugin-root>/skills/`.

## Skills

Each skill lives in `plugins/<name>/skills/<name>/SKILL.md` and contains:

- A YAML front-matter block with `name`, `description`, and trigger phrases
- Detailed instructions Claude follows when the skill is invoked

| Skill | Category | Description |
|-------|----------|-------------|
| [commit](plugins/commit/skills/commit/SKILL.md) | vcs | Create conventional commit messages (feat, fix, docs, etc.) following the Conventional Commits spec |
| [jj](plugins/jj/skills/jj/SKILL.md) | vcs | All version control operations using the Jujutsu (`jj`) CLI — commits, bookmarks, rebasing, workspaces |
| [new-feature](plugins/new-feature/skills/new-feature/SKILL.md) | planning | Scaffold a new feature: gather requirements, create an isolated workspace, and produce a development plan before writing code |
| [pr](plugins/pr/skills/pr/SKILL.md) | vcs | Create a pull request for the current branch with a structured description, using a repo template if one exists |
| [tbd](plugins/tbd/skills/tbd/SKILL.md) | planning | Plan and implement features as a stack of short-lived, independently-green PRs using trunk-based development |

## Adding a New Skill

1. Create `plugins/<name>/skills/<name>/SKILL.md` with a YAML front-matter block (`name`, `description`) and skill body.
2. Add an entry to `.claude-plugin/marketplace.json` under `plugins`:
   ```json
   {
     "name": "<name>",
     "description": "<short description>",
     "source": "./plugins/<name>",
     "version": "1.0.0",
     "category": "<category>",
     "tags": ["<tag1>", "<tag2>"]
   }
   ```
3. Add a row to the tables in `README.md` and `CLAUDE.md`.
