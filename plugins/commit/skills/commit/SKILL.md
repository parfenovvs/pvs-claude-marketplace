---
name: commit
description: >
  Use when the user wants to create a commit message or commit changes.
  Follows Conventional Commits specification with feat, fix, ci, chore, docs,
  style, refactor, perf, test prefixes. Author-neutral messages, no AI persona.
  Triggers on: commit, write commit message, commit changes, describe change.
---

# Commit Message Skill

Create well-structured, author-neutral commit messages following the Conventional Commits specification.

## Commit Message Format

```
<type>(<optional scope>): <short summary>

<optional body>

<optional footer>
```

## Types

| Type       | When to use                                                         |
|------------|---------------------------------------------------------------------|
| `feat`     | A new feature for the user                                          |
| `fix`      | A bug fix                                                           |
| `docs`     | Documentation changes only                                          |
| `style`    | Formatting, missing semicolons, whitespace — no logic change        |
| `refactor` | Code change that neither fixes a bug nor adds a feature             |
| `perf`     | Performance improvement                                             |
| `test`     | Adding or correcting tests                                          |
| `build`    | Changes affecting the build system or external dependencies         |
| `ci`       | Changes to CI configuration files and scripts                       |
| `chore`    | Maintenance tasks, dependency updates, tooling — no production code |
| `revert`   | Reverts a previous commit                                           |

## Summary Line Rules

- **72 characters max** (including type and scope prefix)
- **Imperative mood**: "add feature" not "added feature" or "adds feature"
- **Lowercase** after the colon
- **No period** at the end
- Describes *what* the change does, not *how* or *why* (that goes in the body)

## Body (when needed)

- Separate from the summary with a blank line
- Wrap at 72 characters per line
- Explain the *what* and *why*, not the *how*
- Use bullet points (`-`) for multiple points

## Footer

- Use for breaking changes: `BREAKING CHANGE: <description>`
- Use for issue references: `Closes #123`, `Fixes #456`
- **Never include AI authorship lines** — no "Generated with", "Co-Authored-By: Claude", or any AI attribution
- If a sign-off is needed, read the user's identity from `jj config get user.name` and `jj config get user.email`, then use: `Signed-off-by: <name> <email>`

## Style Guidelines

- **Author-neutral**: The message describes the change, not who made it or with what tools
- **No AI persona**: Never mention AI tools, Claude, or automated generation anywhere in the message
- **Concise**: Prefer one precise sentence over two vague ones

## Workflow

1. Run `jj diff` to inspect the current working copy changes, or `jj diff -r <rev>` for a specific change
2. If the working copy is empty, run `jj status` and ask the user what to include
3. Analyze the diff to understand the nature and scope of changes
4. Select the appropriate type; if multiple types apply, use the dominant change or propose splitting into separate changes
5. Draft the summary line — imperative mood, lowercase after colon, under 72 chars
6. Add a body only when the *why* is non-obvious from the diff
7. Present the message to the user for approval
8. Apply with `jj describe -m "<message>"` or `jj commit -m "<message>"`

## Examples

```
feat(auth): add OAuth2 login flow with Google provider
```

```
fix(api): handle nil pointer when user session expires

Requests to protected endpoints returned a 500 instead of 401
when the session token was present but the backing store had
already evicted the entry.
```

```
refactor(db): extract query builder into separate package
```

```
chore: upgrade dependencies to latest patch versions
```

```
ci: add matrix build for Go 1.21 and 1.22
```

```
feat(payments)!: replace Stripe v2 with v3 SDK

BREAKING CHANGE: PaymentIntent now requires an idempotency key.
Clients must pass X-Idempotency-Key on all POST /payments requests.
```

## Anti-patterns to Avoid

- `fix stuff` — too vague
- `WIP` — not a finished commit message
- `Update code` — says nothing
- `feat: Added new button` — past tense, capital after colon
- `fix: fixed the bug that was causing issues with the login page when users tried to log in using their email address on mobile devices` — over 72 chars, verbose
- Any mention of "Claude", "AI", "generated", "co-authored by AI", etc.
