---
name: pr
description: Create a pull request for the current branch. Searches for a PR template in the project and uses it to structure the PR description.
---

# Create Pull Request

Create a pull request for the current branch with a well-structured description.

## Workflow

### 1. Gather Branch Context

Run these in parallel to understand the current state:

```bash
git status
git branch --show-current
git log main..HEAD --oneline   # or master..HEAD if main doesn't exist
git diff main...HEAD --stat
```

If the branch has no commits ahead of the base branch, stop and inform the user there is nothing to PR.

### 2. Search for PR Template

Search for a pull request template in the following locations (in order):

```
.github/pull_request_template.md
.github/PULL_REQUEST_TEMPLATE.md
.github/PULL_REQUEST_TEMPLATE/*.md   (pick the most relevant one)
docs/pull_request_template.md
PULL_REQUEST_TEMPLATE.md
```

Use the Glob tool to search:

```
.github/pull_request_template.md
.github/PULL_REQUEST_TEMPLATE.md
.github/PULL_REQUEST_TEMPLATE/**/*.md
docs/pull_request_template.md
pull_request_template.md
```

**If a template is found:** Read it and use its structure as the basis for the PR description. Fill in all sections with relevant information from the commits and diff. Remove any sections that are not applicable and placeholder comments.

**If no template is found:** Use the default structure below.

### 3. Determine Base Branch

```bash
git remote show origin | grep "HEAD branch"
```

Use the detected default branch (`main`, `master`, `develop`, etc.) as the base. Fall back to `main` if detection fails.

### 4. Compose PR Title and Description

**Title:** Short (under 72 characters), imperative mood, describes the change. Do not prefix with ticket numbers unless the template requires it.

**Default description structure** (when no template exists):

```markdown
## Summary

<1–3 bullet points explaining what changed and why>

## Changes

<bullet list of key changes grouped by area if needed>

## Test plan

<checklist of how to verify the changes work>
```

Populate every section with concrete details from the commits and diff. Do not leave placeholder text.

### 5. Push Branch if Needed

Check if the current branch has a remote tracking branch:

```bash
git rev-parse --abbrev-ref --symbolic-full-name @{u} 2>/dev/null || echo "no-upstream"
```

If there is no upstream, push the branch first:

```bash
git push -u origin <branch-name>
```

### 6. Create the Pull Request

```bash
gh pr create \
  --title "<title>" \
  --body "$(cat <<'EOF'
<description>
EOF
)" \
  --base <base-branch>
```

If `gh` is not authenticated or not installed, output the title and description so the user can create the PR manually.

### 7. Report Result

Output the PR URL so the user can review it. If the PR was created successfully, confirm the base branch, title, and link.
