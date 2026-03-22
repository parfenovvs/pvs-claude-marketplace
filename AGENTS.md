# ai-hub

Personal collection of AI configs and Claude Code skills.

## Purpose

This repository stores reusable [Claude Code](https://claude.ai/code) skills — prompt-driven behaviours that extend what Claude can do in a project. Drop the `skills/` directory (or symlink individual skill directories) into any project to make the skills available.

## Repository Structure

```
ai-hub/
├── README.md          # Overview and skills table
├── AGENTS.md          # This file — project structure and guidance for AI agents
├── CLAUDE.md          # Symlink to AGENTS.md
└── skills/            # Claude Code skill definitions
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

## Skills

Each skill lives in `skills/<name>/SKILL.md` and contains:

- A YAML front-matter block with `name`, `description`, and trigger phrases
- Detailed instructions Claude follows when the skill is invoked

| Skill | Description |
|-------|-------------|
| [commit](skills/commit/SKILL.md) | Create conventional commit messages (feat, fix, docs, etc.) following the Conventional Commits spec |
| [jj](skills/jj/SKILL.md) | All version control operations using the Jujutsu (`jj`) CLI — commits, bookmarks, rebasing, workspaces |
| [new-feature](skills/new-feature/SKILL.md) | Scaffold a new feature: gather requirements, create an isolated workspace, and produce a development plan before writing code |
| [pr](skills/pr/SKILL.md) | Create a pull request for the current branch with a structured description, using a repo template if one exists |
| [tbd](skills/tbd/SKILL.md) | Plan and implement features as a stack of short-lived, independently-green PRs using trunk-based development |

## Adding a New Skill

1. Create `skills/<name>/SKILL.md`.
2. Add a YAML front-matter block with at minimum `name` and `description`.
3. Write the skill body: workflow steps, rules, examples.
4. Add a row to the tables in `README.md` and `AGENTS.md`.
