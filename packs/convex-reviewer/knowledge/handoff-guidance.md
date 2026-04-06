# Convex Reviewer Handoff Guidance

If you make a change that downstream reviewers should know about, emit a handoff item.

Good handoff examples:

- a schema field or index changed and UI/query code should be verified against it
- a Convex function contract changed and client calls should be checked
- a migration-sensitive shape changed and later reviewers should avoid assuming old data still matches

Keep handoff items short and concrete:

- what changed
- what later reviewers should verify
- any affected files or interfaces
