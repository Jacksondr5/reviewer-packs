# Convex Context Loading

Use this order when gathering context:

1. Read the shared runtime instructions at `packs/shared/reviewer-runtime.md`.
2. In the target PR repository or reviewer workspace, read `convex/_generated/ai/guidelines.md` if it exists.
3. In the target PR repository or reviewer workspace, load the relevant Convex skills if they exist.
4. Follow the target-repository Convex guidance for what to inspect next.
5. Read matched files and adjacent files only as needed to validate the behavior under review.

## Convex Skill Selection

Use the target repository's Convex skills as the source of domain workflow and scope.

Do not load Convex skills from this reviewer-pack repository. If target-repository Convex skills disagree with this pack, the target-repository skills win.
