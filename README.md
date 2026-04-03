# pvs-claude-marketplace — Claude Code Plugin Marketplace

A personal collection of reusable [Claude Code](https://claude.ai/code) skills, distributed as a plugin marketplace.

## Install

Add this marketplace to Claude Code:

```
/plugin marketplace add parfenovvs/pvs-claude-marketplace
```

Then install individual plugins:

```
/plugin install commit@pvs-claude-marketplace
```

## Plugins

| Plugin | Category | Description |
|--------|----------|-------------|
| [commit](plugins/commit/skills/commit/SKILL.md) | vcs | Create conventional commit messages (feat, fix, docs, etc.) following the Conventional Commits spec |
| [jj](plugins/jj/skills/jj/SKILL.md) | vcs | All version control operations using the Jujutsu (`jj`) CLI — commits, bookmarks, rebasing, workspaces |
| [pr](plugins/pr/skills/pr/SKILL.md) | vcs | Create a pull request for the current branch with a structured description, using a repo template if one exists |
| [new-feature](plugins/new-feature/skills/new-feature/SKILL.md) | planning | Scaffold a new feature: gather requirements, create an isolated workspace, and produce a development plan before writing code |
| [tbd](plugins/tbd/skills/tbd/SKILL.md) | planning | Plan and implement features as a stack of short-lived, independently-green PRs using trunk-based development |
| [vcs-workflow](plugins/vcs-workflow/skills/vcs-workflow/SKILL.md) | vcs | Unified VCS workflow: jj for local operations, gh for GitHub, trunk-based development with stacked PRs, and error recovery |

## Marketplace catalog

The full catalog is defined in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
