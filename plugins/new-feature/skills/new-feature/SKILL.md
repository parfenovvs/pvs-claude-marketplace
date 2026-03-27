---
name: new-feature
description: >
  Use when the user wants to start implementing a new feature. Accepts a
  feature description (plain text) or a URL to a requirements document, issue,
  or ticket. Analyses the requirements, creates an isolated workspace for the
  feature, plans the development, and presents the plan for user approval
  before any code is written. Triggers on: new feature, implement feature,
  start feature, feature request, build feature, /new-feature.
---

# New Feature Skill

Orchestrate the start of a new feature: gather requirements, understand the
project, set up an isolated workspace, and produce a development plan — all
before writing a single line of code.

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
- If ambiguous or underspecified, ask up to three targeted clarifying questions.
  Do not ask for information that can be inferred from the codebase.

### 2. Understand the Project

Read enough of the codebase to determine:
- Which layers/modules will be touched.
- Existing patterns to follow (naming, file layout, test conventions).
- **Which VCS and workflow tools the project uses.** Look at config files,
  dotfiles, CI scripts, and any developer documentation — not just whether
  `.git` exists. The project may use git under a wrapper, an alternate VCS
  (jj, Mercurial, etc.), or have specific branching conventions. Identify the
  actual tools in use before creating any workspace.

### 3. Create an Isolated Workspace

Derive a short name for the workspace from the feature (kebab-case, ≤ 30
chars). Then use the appropriate skill or approach for the detected VCS to
create an isolated workspace or worktree. Follow the project's own conventions
for where and how feature work is isolated.

### 4. Plan the Feature

Apply the `/tbd` skill's decomposition and planning approach to break the
feature into an ordered stack of independently-green PRs. Use that skill's
planning workflow and stack plan template to structure the output.

### 5. Present the Plan

Output the plan and wait for explicit user approval before writing any code or
making any commits. Ask: **"Does this plan look right? Any changes before I
start implementing?"**

---

## After Approval

Implement one PR at a time from the bottom of the stack up. For each PR:
write code → run tests → commit → use `/pr` to open the pull request → wait
for it to merge before starting the next.

If requirements shift mid-stack, re-present an updated plan for approval before
continuing.

---

## Anti-Patterns

- Writing code before the plan is approved.
- Creating the workspace after starting to code.
- Assuming the VCS from `.git` alone without checking project tooling.
- Duplicating planning or VCS logic instead of delegating to the relevant skill.
- Introducing a feature flag without a removal PR already in the plan.
