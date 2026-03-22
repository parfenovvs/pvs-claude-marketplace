---
name: tbd
description: >
  Use when the user wants to plan or implement a feature using trunk-based
  development (TBD) with short-lived feature branches and stacked PRs.
  Guides decomposition of features into an ordered PR stack where each PR
  is independently green. Triggers on: trunk-based development, TBD,
  feature branch, stacked PRs, plan feature with TBD, PR stack.
---

# Trunk-Based Development Skill

Plan and implement features as a stack of short-lived, independently-green
pull requests that integrate into trunk frequently.

## Non-Negotiables

- **Trunk is always deployable** — no PR merges if it breaks the build, fails
  tests, or regresses UX
- **Each PR is independently green** — builds pass, tests pass, UX is intact,
  even if the feature is not yet fully visible to users
- **Plan the stack before writing code** — map every PR in the stack upfront;
  do not discover dependencies mid-implementation

---

## The Green Gate Rule

Every PR in a stack must satisfy all three independently:

1. **Build**: compiles/lints without errors
2. **Tests**: all existing tests pass; new behaviour has tests
3. **UX**: no visible regression — use feature flags to hide incomplete work

If any gate fails, the PR does not merge, and dependent PRs do not merge either.

---

## Decomposition Strategy

Break a feature into layers, ordered from lowest dependency to highest:

```
1. Infrastructure / schema    — migrations, new tables, config, env vars
2. Data / API layer           — service methods, API endpoints, data access
3. Business logic             — rules, validation, transformations
4. UI / presentation          — components, pages, wiring
5. Integration / polish       — end-to-end wiring, feature flag removal, cleanup
```

Rules for splitting:
- Each PR should change ~200 lines or fewer (excluding generated files)
- One logical concern per PR — reviewers should understand it without the rest
  of the stack
- If a PR requires a change in a lower layer to be testable, split there first
- Prefer vertical slices (infra + logic + UI for one sub-feature) over
  horizontal layers when the sub-feature is small enough to fit in one PR

---

## Feature Flags

Use a feature flag when:
- The UI ships before the backend is complete
- A behaviour change would be visible to users before the stack is fully merged
- The feature needs a controlled rollout

Structure: wrap incomplete paths behind a flag from PR 1; remove the flag in
the final PR of the stack.

---

## Planning Workflow

When a user requests a feature using TBD:

1. **Understand the full scope** — ask clarifying questions until the feature
   boundary is clear
2. **Identify layers** — list the system components touched (DB, API, UI, etc.)
3. **Draft the stack** — produce a numbered list of PRs with:
   - A one-line description of what the PR does
   - What it tests
   - Whether it needs a feature flag
   - Its dependency (which PR must merge first)
4. **Validate green-gate compliance** — for each PR, confirm build + test + UX
   are independently satisfiable
5. **Present the stack to the user for approval** before writing any code
6. **Implement one PR at a time** — start from PR 1 (lowest layer), get it
   merged, then move to PR 2

---

## VCS Workflow for a Stack

Each PR corresponds to a branch built on top of the previous one, not on trunk directly.

When working on a stack:
- Create each branch from the tip of the previous PR's branch, not from trunk
- Push each branch independently so it can be reviewed and merged on its own
- When a lower PR merges into trunk, rebase the remaining branches onto the updated trunk before continuing
- Keep your view of the full stack visible — know which branches are in flight and their order at all times

---

## Stack Plan Template

Present this to the user when planning a feature:

```
Stack: <feature name>

PR 1 — <title>
  Changes : <what files/systems>
  Tests   : <what is tested>
  Gate    : build ✓ / tests ✓ / UX ✓ (flag: none | <flag-name>)

PR 2 — <title>
  Depends : PR 1
  Changes : ...
  Tests   : ...
  Gate    : build ✓ / tests ✓ / UX ✓ (flag: none | <flag-name>)

...

PR N — remove feature flag / final wiring
  Depends : PR N-1
  Gate    : build ✓ / tests ✓ / UX ✓ (flag: removed)
```

---

## Anti-Patterns to Avoid

- **One giant PR for the whole feature** — impossible to review, high merge
  conflict risk, violates TBD
- **PRs that only pass tests when stacked** — each PR must be green on its own
- **Skipping the plan** — starting implementation before the stack is mapped
  leads to mid-stack redesigns
- **Merging a red PR to unblock the next** — this spreads the failure across
  the stack; fix the gate failure first
- **Feature flag sprawl** — every flag must have a removal PR already planned
  in the stack before the flag is introduced
