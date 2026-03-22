# ai-hub

Claude Code plugin marketplace — personal collection of skills distributed via the Claude Code plugin system.

## Purpose

This repository is a [Claude Code plugin marketplace](https://code.claude.com/docs/en/plugin-marketplaces). It hosts skills that users install with `/plugin install <name>@ai-hub`. Each skill is a prompt-driven behaviour that extends what Claude can do in a project.

## Repository Structure

```
ai-hub/
├── README.md                        # Marketplace overview and install instructions
├── CLAUDE.md                        # This file — project structure and guidance for AI agents
├── .claude-plugin/
│   └── marketplace.json             # Marketplace catalog (required by Claude Code)
└── skills/                          # One directory per skill/plugin
    ├── commit/        # Conventional commit message authoring
    │   └── SKILL.md
    ├── jj/            # Jujutsu (jj) version control operations
    │   └── SKILL.md
    ├── new-feature/   # New feature scaffolding and planning
    │   └── SKILL.md
    ├── pr/            # Pull request creation
    │   └── SKILL.md
    └── tbd/           # Trunk-based development planning
        └── SKILL.md
```

## Marketplace Catalog

`.claude-plugin/marketplace.json` is the entry point for the Claude Code plugin system. It lists every plugin in this repo, their sources, categories, and tags. Claude Code reads this file when users run `/plugin marketplace add` or `/plugin install`.

Each plugin entry references skills in `skills/<name>/` using `"source": "./"` and `"strict": false` so the marketplace controls component exposure rather than requiring per-plugin `plugin.json` files.

## Skills

Each skill lives in `skills/<name>/SKILL.md` and contains:

- A YAML front-matter block with `name`, `description`, and trigger phrases
- Detailed instructions Claude follows when the skill is invoked

| Skill | Category | Description |
|-------|----------|-------------|
| [commit](skills/commit/SKILL.md) | vcs | Create conventional commit messages (feat, fix, docs, etc.) following the Conventional Commits spec |
| [jj](skills/jj/SKILL.md) | vcs | All version control operations using the Jujutsu (`jj`) CLI — commits, bookmarks, rebasing, workspaces |
| [new-feature](skills/new-feature/SKILL.md) | planning | Scaffold a new feature: gather requirements, create an isolated workspace, and produce a development plan before writing code |
| [pr](skills/pr/SKILL.md) | vcs | Create a pull request for the current branch with a structured description, using a repo template if one exists |
| [tbd](skills/tbd/SKILL.md) | planning | Plan and implement features as a stack of short-lived, independently-green PRs using trunk-based development |

## Adding a New Skill

1. Create `skills/<name>/SKILL.md` with a YAML front-matter block (`name`, `description`) and skill body.
2. Add an entry to `.claude-plugin/marketplace.json` under `plugins`:
   ```json
   {
     "name": "<name>",
     "description": "<short description>",
     "source": "./",
     "strict": false,
     "skills": ["./skills/<name>"],
     "version": "1.0.0",
     "category": "<category>",
     "tags": ["<tag1>", "<tag2>"]
   }
   ```
3. Add a row to the tables in `README.md` and `CLAUDE.md`.
