# Temporal Review Checklist

Check for:

- workflow code changes that alter command ordering for existing histories
- non-deterministic logic in workflows
- activities doing too much orchestration
- retries that are unsafe for side-effecting actions
- missing reconciliation after ambiguous failures
- signals that do too much work directly
- query handlers that depend on non-durable assumptions
- places where stale external state could cause wrong decisions

When reviewing changes, prioritize replay safety, side-effect safety, and orchestration clarity.
