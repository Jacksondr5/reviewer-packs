# Convex Context Loading

Use this order when gathering context:

1. Read `convex/_generated/ai/guidelines.md` in the target repository.
2. Read `convex/schema.ts`.
3. Infer intent and load the matching Convex skill and reference docs if applicable:
   - Auth/provider changes → `.agents/skills/convex-setup-auth/SKILL.md` plus the matching provider reference
   - Migration/scheme-breaking data changes → `.agents/skills/convex-migration-helper/SKILL.md`
   - Component extraction/boundary work → `.agents/skills/convex-create-component/SKILL.md`
   - Query/mutation slowness or contention → `.agents/skills/convex-performance-audit/SKILL.md`
4. Read the Convex functions and touched `convex/**/*.ts` files.
5. Read any UI/orchestrator code only if needed to validate contracts and usage.

## What Matters Most

- whether schema fields and validators match actual usage
- whether indexes exist for the intended query patterns
- whether queries avoid anti-patterns like broad filtering
- whether mutations/actions are structured safely
- whether high-churn and stable data are separated appropriately
- whether migration-safe rollout patterns are needed for breaking shape changes
- whether component boundaries are respected where components are used (wrapper functions, ID crossing, auth/env location)

## What Matters Less

- generic React styling or component structure unless it is directly coupled to Convex behavior
- generic TypeScript style issues unrelated to Convex correctness
