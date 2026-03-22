# ai-hub — Claude Code Plugin Marketplace

A personal collection of reusable [Claude Code](https://claude.ai/code) skills, distributed as a plugin marketplace.

## Install

Add this marketplace to Claude Code:

```
/plugin marketplace add parfenovvs/ai-hub
```

Then install individual plugins:

```
/plugin install commit@ai-hub
```

## Plugins

| Plugin | Category | Description |
|--------|----------|-------------|
| [commit](skills/commit/SKILL.md) | vcs | Create conventional commit messages (feat, fix, docs, etc.) following the Conventional Commits spec |
| [jj](skills/jj/SKILL.md) | vcs | All version control operations using the Jujutsu (`jj`) CLI — commits, bookmarks, rebasing, workspaces |
| [pr](skills/pr/SKILL.md) | vcs | Create a pull request for the current branch with a structured description, using a repo template if one exists |
| [new-feature](skills/new-feature/SKILL.md) | planning | Scaffold a new feature: gather requirements, create an isolated workspace, and produce a development plan before writing code |
| [tbd](skills/tbd/SKILL.md) | planning | Plan and implement features as a stack of short-lived, independently-green PRs using trunk-based development |

## Marketplace catalog

The full catalog is defined in [`.claude-plugin/marketplace.json`](.claude-plugin/marketplace.json).
