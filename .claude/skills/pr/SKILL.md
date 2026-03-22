---
name: pr
description: Create a pull request for the current branch. Searches for a PR template in the project and uses it to structure the PR description.
---

# Create Pull Request

Create a pull request for the current branch with a well-structured description.

## Workflow

### 1. Gather Branch Context

Determine the current branch, the appropriate base branch (e.g. `main`, `master`, `develop`), the commits that will be included in the PR, and a summary of changed files.

If the branch has no commits ahead of the base branch, stop and inform the user there is nothing to PR.

### 2. Search for PR Template

Search for a pull request template in the project. Common locations to check:

- `.github/pull_request_template.md`
- `.github/PULL_REQUEST_TEMPLATE.md`
- `.github/PULL_REQUEST_TEMPLATE/` (directory with multiple templates — pick the most relevant)
- `docs/pull_request_template.md`
- `pull_request_template.md`

**If a template is found:** Use its structure as the basis for the PR description. Fill in all sections with relevant information from the commits and diff. Remove inapplicable sections and placeholder comments.

**If no template is found:** Use the default structure below.

### 3. Compose PR Title and Description

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

### 4. Ensure Branch is Published

If the current branch has not been pushed to the remote yet, push it so the PR can be created against it.

### 5. Create the Pull Request

**If `gh` (GitHub CLI) is available and the remote is GitHub:**

Use `gh pr create` with the composed title, description, and base branch. If `gh` is not authenticated, output the title and description so the user can open the PR manually.

**For other VCS platforms (GitLab, Bitbucket, Azure DevOps, etc.):**

Output the composed title and description in full so the user can paste them into the platform's PR/MR creation UI. Include a note on the base branch to target.

### 6. Report Result

If the PR was created via CLI, return the PR URL. Otherwise, confirm the composed content was printed and remind the user which base branch to target.
