# Temporal Reviewer Handoff Guidance

If you make a change that downstream reviewers should know about, emit a handoff item.

Good handoff examples:

- workflow state shape changed and UI/debug surfaces should reflect it
- activity outputs changed and persistence/rendering should be verified
- retry/recovery semantics changed and later reviewers should avoid stale assumptions

Keep handoff items short and concrete:

- what changed
- what later reviewers should verify
- any affected workflow/activity/state files
