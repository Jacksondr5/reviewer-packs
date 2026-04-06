# Temporal Context Loading

Use this order when gathering context:

1. Read the workflow entrypoints and their state/domain models.
2. Read the activities invoked by the workflow path you are reviewing.
3. Read signal/query definitions and any reconciliation helpers.
4. Read docs or design notes only as needed to understand intended behavior.

## What Matters Most

- whether workflow code stays deterministic
- whether changes to workflow command ordering can break replay
- whether activities own side effects and workflows own orchestration
- whether retries/timeouts match the side-effect profile
- whether signals/queries are handled safely

## What Matters Less

- generic TypeScript style issues unrelated to Temporal behavior
- UI or API shape unless it changes orchestration semantics
