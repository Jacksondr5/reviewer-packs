# Temporal Context Loading

Use this order when gathering context:

1. Read the shared runtime instructions at `packs/shared/reviewer-runtime.md`.
2. In the target PR repository or reviewer workspace, load the relevant Temporal skills if they exist.
3. Follow the target-repository Temporal guidance for what to inspect next.
4. Read matched files and adjacent files only as needed to validate the behavior under review.

## Temporal Skill Reference Selection

Use the target repository's Temporal skills as the source of domain workflow and scope.

Do not load Temporal skills from this reviewer-pack repository. If target-repository Temporal skills disagree with this pack, the target-repository skills win.
