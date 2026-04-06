# Convex Review Checklist

Check for:

- schema changes that need migration planning
- missing or incorrect validators on functions and table fields
- missing indexes for newly introduced queries or scans
- unbounded reads: `.collect()` on tables without limits, or scans over growing data
- broad scan + filter patterns:
  - JS post-filtering after wide reads
  - Convex `query.filter(...)` used as a substitute for a real index or search index
- function boundary and runtime safety:
  - action/query/mutation misuse
  - unnecessary `ctx.runQuery`/`ctx.runMutation` inside the same module where direct helpers apply
  - use of actions where a transaction-aware query/mutation is required (and vice versa)
- tables accumulating unrelated concerns or unbounded embedded arrays
- auth-sensitive functions that trust caller-supplied identity arguments
- UI/server changes that assume fields or query args changed without contract updates
- component boundary hazards if components are present:
  - component-local schema using parent `Id` types directly
  - missing app-level wrappers for auth/env or public access

When reviewing changes, prefer precise, actionable findings over broad stylistic commentary.
