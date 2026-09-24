# Git Workflow

Git history should be small, coherent and traceable.

## Commits

Changes should be committed in small, logical units.

Each commit should:

- represent one coherent change
- have a clear and meaningful purpose
- avoid unrelated modifications
- be small enough to review and understand independently
- leave the application in a valid state whenever practical

Do not combine unrelated features, refactorings, formatting changes or
documentation changes into a single commit merely for convenience.

Prefer several focused commits over one large commit.

Commits should make it possible to understand what changed and why
without reconstructing the entire development session.

Do not create commits merely to record intermediate agent activity.

## Branches

Before making changes, the agent must determine whether the user wants
the work performed on the current branch or in a separate branch.

If this has not already been specified, ask the user:

> Ska förändringarna göras i den aktuella branchen eller i en separat branch?

Do not create, switch to, rename or delete branches without the user's
explicit instruction.

Once the branch decision has been made, keep all related changes within
that branch unless the user instructs otherwise.

## Scope

A single task may result in multiple commits when the changes represent
distinct logical steps.

For example:

```text
feat: add room approval flow
test: add room approval scenarios
docs: update room user cases
```

Do not artificially split a single inseparable change merely to increase
the number of commits.

## Before Committing

Before creating a commit:

- Inspect the changes.
- Verify that unrelated changes are not included.
- Run the relevant tests and validation.
- Confirm that the commit represents the intended change.
- Use a clear commit message describing the change.

The agent must not commit unrelated user changes that were already present
in the working tree.

## User Changes

Never discard, overwrite or silently modify uncommitted user changes.

If existing changes make the requested work ambiguous or unsafe, stop and
ask the user before proceeding.
