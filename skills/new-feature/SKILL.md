---
name: new-feature
description: >
  Use when the user wants to start implementing a new feature. Accepts a
  feature description (plain text) or a URL to a requirements document, issue,
  or ticket. Analyses the requirements, creates an isolated workspace for the
  feature, plans the development as a stack of PRs, and presents the plan for
  user approval before any code is written. Triggers on: new feature, implement
  feature, start feature, feature request, build feature, /new-feature.
---

# New Feature Skill

Analyse requirements, set up an isolated workspace, and produce a reviewed
development plan — before a single line of code is written.

---

## Inputs

The user provides one of:
- **Plain-text description** — a sentence or paragraph describing the feature.
- **URL** — a link to a GitHub issue, Linear ticket, Notion doc, Confluence
  page, Jira ticket, or any other requirements source.

---

## Workflow

### 1. Gather Requirements

**If the input is a URL:**
- Fetch the page content.
- Extract: goal/objective, user stories or acceptance criteria, edge cases,
  any referenced designs or related issues.
- If the URL is behind authentication and cannot be fetched, ask the user to
  paste the relevant content directly.

**If the input is plain text:**
- Use the description as-is.
- If the description is ambiguous or underspecified, ask up to three targeted
  clarifying questions before proceeding. Do not ask for information that can
  be inferred from the codebase.

### 2. Explore the Codebase

Before planning, read enough of the codebase to understand:
- Which layers/modules will be touched (schema, API, business logic, UI, etc.).
- Existing patterns to follow (naming, file layout, test conventions).
- Any related code that the feature builds on or modifies.

### 3. Detect VCS Type

Check whether the repository uses **jj** (Jujutsu) or plain **git**:

```bash
# jj repo: .jj/ directory exists at repo root
ls .jj 2>/dev/null && echo "jj" || echo "git"
```

### 4. Create an Isolated Workspace

**Derive a short workspace name** from the feature (kebab-case, max 30 chars,
no special characters beyond hyphens).

#### jj repository

Create a jj workspace so the feature is isolated from the main working copy:

```bash
# From the repo root
jj workspace add .jj/workspace/<feature-name>
cd .jj/workspace/<feature-name>

# Confirm isolation
jj status   # @ is this workspace's working copy, independent of main
```

#### git repository

Create a git worktree on a new branch:

```bash
git worktree add .git/worktrees/<feature-name> -b feat/<feature-name>
cd .git/worktrees/<feature-name>

# Confirm
git branch   # should show feat/<feature-name> checked out
```

After creating the workspace, `cd` into it so all subsequent VCS commands
operate there.

### 5. Plan the Feature

Decompose the feature into an ordered stack of PRs following trunk-based
development principles. Apply the decomposition layers:

```
1. Infrastructure / schema    — migrations, new tables, config, env vars
2. Data / API layer           — service methods, API endpoints, data access
3. Business logic             — rules, validation, transformations
4. UI / presentation          — components, pages, wiring
5. Integration / polish       — end-to-end wiring, flag removal, cleanup
```

Rules:
- Each PR should be independently green (builds, tests pass, no UX regression).
- Use feature flags to hide incomplete work from users.
- Prefer vertical slices for small sub-features; use horizontal layers for
  large ones.
- Target ~200 changed lines per PR (excluding generated files).

### 6. Present the Plan

Output the plan using the template below. **Do not write any code or make any
VCS commits until the user approves the plan.**

---

## Plan Template

```
Feature: <feature name>
Workspace: <path to workspace or worktree>

Requirements summary:
  <2-4 bullet points distilled from the requirements>

Affected areas:
  <bullet list of modules, layers, or files that will change>

─────────────────────────────────────────────
PR Stack
─────────────────────────────────────────────

PR 1 — <title>
  Branch  : feat/<feature-name>/pr-1-<slug>
  Changes : <what files/systems>
  Tests   : <what is tested>
  Gate    : build ✓ / tests ✓ / UX ✓  (flag: none | <flag-name>)

PR 2 — <title>
  Branch  : feat/<feature-name>/pr-2-<slug>
  Depends : PR 1
  Changes : ...
  Tests   : ...
  Gate    : build ✓ / tests ✓ / UX ✓  (flag: none | <flag-name>)

...

PR N — Remove feature flag / final wiring
  Branch  : feat/<feature-name>/pr-N-<slug>
  Depends : PR N-1
  Gate    : build ✓ / tests ✓ / UX ✓  (flag: removed)

─────────────────────────────────────────────
Open questions (if any):
  - <anything that needs user clarification before implementation starts>
─────────────────────────────────────────────
```

After presenting the plan, ask: **"Does this plan look right? Any changes
before I start implementing?"** Wait for explicit approval before writing code.

---

## After Approval

Once the user approves:

1. Implement **one PR at a time**, starting from PR 1.
2. Within each PR: write code → run tests → commit → push → open PR (use the
   `/pr` skill to create the pull request).
3. Wait for the PR to merge before starting the next one.
4. If requirements shift mid-stack, re-present an updated plan for approval
   before continuing.

---

## Anti-Patterns to Avoid

- Skipping the plan and writing code immediately.
- Creating the workspace after starting to code.
- Starting PR 2 before PR 1 is merged.
- Letting a PR grow beyond its defined scope.
- Introducing a feature flag without a removal PR already in the plan.
- Fetching private URLs without warning the user that content will be sent
  over the network.
