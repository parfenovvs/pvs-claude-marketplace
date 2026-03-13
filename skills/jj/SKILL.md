---
name: jj
description: >
  Use for ALL version control operations in projects. Handles commits, history,
  branches (bookmarks), remotes, diffs, conflict resolution, workspaces, and
  repository management using the jujutsu (jj) CLI. Triggers on: commit, branch,
  push, pull, merge, diff, log, history, undo, stash, rebase, squash, worktree,
  workspace, or any VCS/source control request.
---

# Jujutsu (jj) Version Control Skill

Use `jj` for all version control operations. Never fall back to raw `git` commands
for repository operations — `jj` manages the underlying git repo transparently.

## Core Concepts

### Changes vs Commits
- A **change** is the primary unit in jj (like a git commit, but mutable)
- Changes have a stable **change ID** (e.g. `qpvuntsm`) that persists even after rebasing
- Changes also have a **commit ID** (content hash) that changes when content changes
- Use the **shortest unique prefix** of a change ID in commands (e.g. `qp` instead of full ID)

### Working Copy (`@`)
- The current working copy is always a change, denoted by `@` in logs
- jj **auto-snapshots** the working copy at the start of every command — no `git add` needed
- All files are tracked by default; use `.gitignore` to exclude files

### Revsets
A functional query language for selecting commits:
```
@               # current working copy
@-              # parent of working copy
trunk           # bookmark named "trunk"
main            # bookmark named "main"
trunk..@        # all changes from trunk up to @
roots(trunk..@) # root changes in that range (no parent in the set)
heads(trunk..@) # head changes in that range (no child in the set)
all:X           # prefix for commands expecting multiple revisions
```

### Bookmarks (Branches)
- Bookmarks are named pointers to commits (equivalent to git branches)
- They do **not** move automatically — must be explicitly updated
- Remote tracking: `main@origin` tracks the remote bookmark

---

## Command Reference

### Status & Inspection

```bash
jj status            # Show working copy status (alias: jj st)
jj log               # Show commit graph
jj log --limit 10    # Limit to 10 entries
jj log -r trunk..@   # Show range of changes
jj show              # Show current change diff
jj show <rev>        # Show specific change diff
jj diff              # Diff working copy vs parent
jj diff -r <rev>     # Diff a specific change
jj diff --from <rev> --to <rev>  # Diff between two changes
```

### Creating & Editing Changes

```bash
jj new               # Create new empty change on top of @, move @ to it
jj new <parent>      # Create new change with specific parent
jj new --no-edit <parent>  # Create change without moving @
jj describe -m "message"   # Set description on current change
jj describe -r <rev> -m "message"  # Set description on specific change
jj commit -m "message"     # Describe current change and create new empty one on top
```

### Moving & Reorganizing

```bash
jj rebase -s <source> -d <destination>   # Rebase source onto destination
jj rebase -s 'all:roots(trunk..@)' -d trunk  # Rebase all local changes onto trunk
jj squash                                # Squash @ into parent
jj squash -r <rev>                       # Squash revision into its parent
jj squash --from <rev> --into <rev>      # Squash one change into another
jj split                                 # Interactively split current change
jj absorb                                # Move working copy changes into appropriate stack commits
```

### Undo & Abandon

```bash
jj undo              # Undo the last operation (SAFE — jj tracks all operations)
jj redo              # Redo last undone operation
jj abandon <rev>     # Delete a change (descendants are rebased automatically)
jj op log            # Show operation history
jj op undo <op>      # Undo a specific past operation
```

### Bookmarks (Branches)

```bash
jj bookmark list                 # List all bookmarks (alias: jj b l)
jj bookmark create <name>        # Create bookmark at @
jj bookmark create <name> -r <rev>  # Create at specific revision
jj bookmark set <name>           # Move existing bookmark to @
jj bookmark set <name> -r <rev>  # Move bookmark to revision
jj bookmark move <name> --to <rev>  # Move bookmark
jj bookmark delete <name>        # Delete bookmark
jj bookmark rename <old> <new>   # Rename bookmark
jj bookmark track <name>@<remote>   # Start tracking remote bookmark
```

### Working with Remotes (Git)

```bash
jj git clone <url>               # Clone a remote repo
jj git fetch                     # Fetch all remotes
jj git fetch --remote origin     # Fetch specific remote
jj git push                      # Push bookmarks to remote
jj git push --bookmark <name>    # Push specific bookmark
jj git push --change @           # Create/push bookmark for current change
jj git remote add <name> <url>   # Add remote
jj git remote list               # List remotes
```

### Conflict Resolution

```bash
jj resolve                       # Open merge tool on conflicted files
jj resolve --list                # List files with conflicts
jj status                        # Shows conflicts with "C" marker
```

### File Operations

```bash
jj restore <file>                # Restore file from parent (discard changes)
jj restore --from <rev> <file>   # Restore file from specific revision
jj file list                     # List tracked files
```

### Workspaces

Workspaces are jj's alternative to git worktrees — multiple working copies sharing the same repo.
Each workspace has its own `@` (working-copy commit) and can be on a different change.

**Always create workspaces inside the project at `PROJECT_DIR/.jj/workspace/<feature-name>`.**

```bash
# List existing workspaces
jj workspace list

# Add a new workspace (creates directory at the specified path)
jj workspace add .jj/workspace/<feature-name>

# Add workspace starting at a specific revision
jj workspace add .jj/workspace/<feature-name> -r <rev>

# Add workspace with an explicit name (defaults to directory basename)
jj workspace add .jj/workspace/<feature-name> --name <name>

# Work in the new workspace (cd into it, jj commands operate there)
cd .jj/workspace/<feature-name>
jj status    # @ is this workspace's working copy

# View all workspaces and their @ in the log (shown as <workspace>@)
jj log       # shows main@ and <feature-name>@ side by side

# Remove a workspace (from the main workspace or any other)
jj workspace forget <name>

# If a workspace becomes stale (e.g. its @ was abandoned from another workspace)
jj workspace update-stale
```

**Typical workspace workflow:**

```bash
# In main project dir: start a new feature in isolation
jj workspace add .jj/workspace/feature-x
cd .jj/workspace/feature-x

# Do work here; @ is independent from main workspace
jj describe -m "wip: feature x"
jj new

# Meanwhile, back in main workspace (open a new terminal):
cd <PROJECT_DIR>
jj status    # main workspace @ is unaffected

# When feature is done, clean up
cd <PROJECT_DIR>
jj workspace forget feature-x
rm -rf .jj/workspace/feature-x   # optional: remove the directory
```

---

## Common Workflows

### Starting a New Project

```bash
# Initialize new jj repo (git-backed, colocated with git)
jj git init --colocate

# Or clone existing git repo
jj git clone https://github.com/user/repo
```

### Daily Work Loop

```bash
# 1. Check status
jj status

# 2. Make changes to files (jj auto-tracks them)

# 3. Describe what you've done
jj describe -m "add feature X"

# 4. Start next unit of work
jj new

# 5. View history
jj log
```

### Committing Work

```bash
# Option A: describe then new
jj describe -m "implement login"
jj new

# Option B: commit (describe + new in one step)
jj commit -m "implement login"
```

### Working on a Feature (with bookmarks)

```bash
# Create a bookmark for the feature
jj bookmark create feature-x

# Do work, create changes...
jj describe -m "start feature X"
jj new
jj describe -m "finish feature X"

# Move the bookmark to current @ when ready
jj bookmark set feature-x

# Push to remote
jj git push --bookmark feature-x
```

### Stacked Changes / Pull Requests

jj excels at stacked diffs. Each change is independently addressable:

```bash
# Create a stack
jj describe -m "refactor: extract helper"
jj new
jj describe -m "feat: use helper in main"
jj new
jj describe -m "test: add tests for helper"

# View the stack
jj log -r trunk..@

# Rebase entire stack onto updated trunk
jj git fetch
jj rebase -s 'all:roots(trunk..@)' -d trunk@origin
```

### Undoing Mistakes

```bash
# Undo last operation (safe — nothing is lost)
jj undo

# View full operation log
jj op log

# Undo a specific operation
jj op undo <op-id>

# Abandon a specific unwanted change
jj abandon <change-id>
```

### Squashing/Cleaning Up Before Push

```bash
# Squash current change into parent
jj squash

# Squash interactively (pick which hunks to squash)
jj squash -i

# Combine multiple changes into one
jj squash --from <child> --into <parent>
```

### Syncing with Remote

```bash
# Fetch latest
jj git fetch

# Rebase local work on top of updated remote main
jj rebase -s 'all:roots(main@origin..@)' -d main@origin

# Push changes
jj git push --change @   # creates a bookmark if needed
```

---

## Key Differences from Git

| Operation | Git | jj |
|-----------|-----|----|
| Stage files | `git add` | Not needed — auto-snapshot |
| Commit | `git commit -m "..."` | `jj commit -m "..."` or `jj describe` + `jj new` |
| Amend last commit | `git commit --amend` | `jj describe` (just edit the current change) |
| Create branch | `git checkout -b name` | `jj bookmark create name` + `jj new` |
| Switch branch | `git checkout name` | `jj edit name` or `jj new name` |
| Undo commit | `git reset HEAD~` | `jj undo` |
| View log | `git log` | `jj log` |
| Stash | `git stash` | Not needed — just `jj new` and come back |
| Interactive rebase | `git rebase -i` | `jj squash`, `jj split`, `jj rebase` |
| Cherry-pick | `git cherry-pick` | `jj duplicate -r <rev>` + rebase |
| Push branch | `git push origin name` | `jj git push --bookmark name` |
| Worktree | `git worktree add` | `jj workspace add .jj/workspace/<name>` |

---

## Safety Notes

- **`jj undo` is always safe** — jj records every operation and any state can be recovered
- **Conflicts are stored in commits** — rebases always succeed; conflicts persist in the change and can be resolved later
- **No staging area** — the working copy IS the current change; `jj new` when ready to start the next one
- **Do not mix raw `git` commands** with `jj` in the same repo — use `jj git import`/`jj git export` if you must interop
