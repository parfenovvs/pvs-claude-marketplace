---
name: vcs-workflow
description: >
  Unified VCS workflow skill: use jj for all local version control, gh for
  GitHub operations (PRs, issues), and trunk-based development with stacked
  PRs. Covers feature branches, daily work loops, PR creation, error recovery,
  and stack management. Triggers on: feature branch, create PR, push, pull,
  stacked PR, trunk-based, vcs workflow, jj workflow, github PR.
---

# VCS Workflow Skill

This skill governs the full version control workflow: **jj** for local repo
operations, **gh** for GitHub interactions, and **trunk-based development**
with stacked PRs.

---

## Tool Responsibilities (Non-Negotiable)

Before every VCS operation, state which tool you are using and why:

| Tool | Scope | Never Use For |
|------|-------|---------------|
| `jj` | All local repo operations: commits, bookmarks, rebasing, diffs, history, workspaces, conflict resolution | Do not use `git` directly |
| `gh` | All GitHub operations: creating/viewing PRs, issues, checks, releases | Do not use `gh` for local repo work |
| `git` | **Never** — `jj` manages the underlying git repo transparently. Only exception: `jj git import`/`jj git export` when explicitly required for interop |

**Always announce the tool before each operation block**, e.g.:

> _Using `jj` to create a new bookmark and snapshot the working copy._
> _Using `gh` to open a pull request against main._

---

## Core jj Concepts

- **Change ID**: stable identifier (e.g. `qpvuntsm`) — persists through rebases. Use shortest unique prefix.
- **`@`**: current working copy — always a change. jj auto-snapshots on every command, no staging needed.
- **Bookmarks**: named pointers to commits (= git branches). Do not move automatically — must be set explicitly.
- **`jj undo` is always safe**: every operation is logged; any state is recoverable via `jj op log`.

---

## Workflow: Feature Branch → Changes → PR

This is the canonical end-to-end workflow. Follow it exactly.

### Step 1 — Sync trunk

> _Using `jj` to fetch remote changes and confirm the working copy is clean._

```bash
jj git fetch
jj log -r 'main@origin'    # confirm trunk is up to date
jj rebase -s 'all:roots(main@origin..@)' -d main@origin  # rebase any local work
```

### Step 2 — Create a feature bookmark

> _Using `jj` to create a named bookmark for the feature branch._

```bash
jj new main@origin              # start a new change on top of remote main
jj bookmark set feature-x       # name the bookmark
jj describe -m "wip: feature x" # optional: label the initial change
```

### Step 3 — Do work, create changes

> _Using `jj` to snapshot work and build the change stack._

```bash
# jj auto-tracks all files — no git add needed
jj describe -m "feat: implement X core logic"
jj new                         # start next change
jj describe -m "feat: wire X into handler"
jj new
jj describe -m "test: add tests for X"
```

View the stack at any time:
```bash
jj log -r 'main@origin..@'
```

### Step 4 — Move the bookmark to the stack tip and push

> _Using `jj` to update the bookmark and push. Using `gh` to open the PR._

```bash
jj bookmark set feature-x      # move bookmark to current @ (tip of stack)
jj git push --bookmark feature-x
```

Then open the PR:
```bash
gh pr create --base main --head feature-x --title "feat: implement X" --body "..."
```

### Step 5 — Respond to review feedback

> _Using `jj` to amend the relevant change in the stack. Using `jj git push` to update the remote branch._

```bash
jj log -r 'main@origin..feature-x'  # identify the change to fix
jj edit <change-id>                  # move @ to that change
# make edits — jj auto-snapshots
jj describe -m "feat: implement X core logic (address review)"
jj git push --bookmark feature-x     # force-updates the remote bookmark
```

### Step 6 — After merge, clean up

> _Using `jj` to abandon the feature bookmark and sync trunk._

```bash
jj git fetch
jj bookmark delete feature-x         # remote is gone after merge
jj abandon 'roots(main@origin..feature-x)'  # clean up local changes if needed
```

---

## Trunk-Based Development with Stacked PRs

Use stacked PRs when a feature is too large for one PR. Each PR must be
**independently green**: build passes, tests pass, no UX regression.

### Stack Decomposition Order

```
1. Infrastructure / schema    — migrations, config, env vars
2. Data / API layer           — endpoints, service methods, data access
3. Business logic             — rules, validation, transformations
4. UI / presentation          — components, wiring
5. Integration / polish       — end-to-end wiring, flag removal, cleanup
```

Rules:
- ~200 lines per PR (excluding generated files)
- One logical concern per PR
- If a PR requires a lower layer to be testable, split there first
- Use feature flags to hide incomplete work; plan the flag-removal PR upfront

### Stack Plan Template

Present before writing any code:

```
Stack: <feature name>

PR 1 — <title>
  Changes : <files/systems>
  Tests   : <what is tested>
  Gate    : build ✓ / tests ✓ / UX ✓  (flag: none | <flag-name>)

PR 2 — <title>
  Depends : PR 1
  Changes : ...
  Tests   : ...
  Gate    : build ✓ / tests ✓ / UX ✓  (flag: none | <flag-name>)

PR N — remove feature flag / final wiring
  Depends : PR N-1
  Gate    : build ✓ / tests ✓ / UX ✓  (flag: removed)
```

### VCS for a Stack

> _Using `jj` to build each PR as a change on top of the previous one._

```bash
# Create the stack (all changes stacked on top of each other)
jj new main@origin && jj bookmark set pr-1-infra
jj new && jj bookmark set pr-2-api     # stacked on pr-1-infra
jj new && jj bookmark set pr-3-ui      # stacked on pr-2-api

# Push all bookmarks
jj git push --bookmark pr-1-infra
jj git push --bookmark pr-2-api
jj git push --bookmark pr-3-ui

# Open PRs: each targets the previous bookmark as base, not main directly
gh pr create --base main  --head pr-1-infra --title "infra: ..."
gh pr create --base pr-1-infra --head pr-2-api --title "api: ..."
gh pr create --base pr-2-api   --head pr-3-ui  --title "ui: ..."
```

After PR 1 merges into main:
```bash
# Using jj to rebase remaining stack onto updated main
jj git fetch
jj rebase -s pr-2-api -d main@origin
jj git push --bookmark pr-2-api
jj git push --bookmark pr-3-ui
```

---

## Error Recovery

### Auth failure on `jj git push`

> _Using `gh` to verify authentication, then retrying push._

```bash
gh auth status                   # check GitHub auth
gh auth login                    # re-authenticate if expired
jj git push --bookmark <name>    # retry
```

### Bookmark conflict (remote has diverged)

> _Using `jj` to inspect the divergence and force-push after confirming intent._

```bash
jj log -r '<bookmark>@origin'    # see what remote has
jj log -r '<bookmark>'           # see what local has
jj diff --from '<bookmark>@origin' --to '<bookmark>'  # inspect difference
# If local is correct:
jj git push --bookmark <name> --allow-new  # or use --force if remote tracking is set
```

### Stale working copy after remote merge

> _Using `jj` to update and rebase local changes._

```bash
jj git fetch
jj log -r 'all()'                # orient yourself
jj rebase -s 'all:roots(main@origin..@)' -d main@origin
```

### Accidentally described the wrong change

```bash
jj undo          # undo the last describe operation — always safe
jj op log        # if you need to go further back
jj op undo <id>  # undo a specific past operation
```

### Merge conflict after rebase

> _Using `jj` to list and resolve conflicts. Conflicts are stored in the change and do not block other operations._

```bash
jj status        # shows "C" markers for conflicted files
jj resolve --list
jj resolve       # open merge tool
# After resolving:
jj describe -m "resolve conflicts with main"
```

### Workspace became stale

```bash
jj workspace update-stale    # fix stale workspace pointer
```

---

## Quick Reference

| Task | Tool | Command |
|------|------|---------|
| Fetch remote | jj | `jj git fetch` |
| Create bookmark | jj | `jj bookmark set <name>` |
| Push bookmark | jj | `jj git push --bookmark <name>` |
| Open PR | gh | `gh pr create ...` |
| View PR status | gh | `gh pr status` / `gh pr checks` |
| Merge PR | gh | `gh pr merge <number>` |
| Undo last op | jj | `jj undo` |
| Rebase stack | jj | `jj rebase -s 'all:roots(main@origin..@)' -d main@origin` |
| Fix conflicts | jj | `jj resolve` |
| Check auth | gh | `gh auth status` |
