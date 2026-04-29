# Reviewer Authoring Guide

This guide explains how to create and maintain specialized reviewer packs.

## Purpose

A specialized reviewer gives the PR review orchestrator focused domain judgment after general remediation has already happened. The reviewer should know what to inspect, what risks matter in its domain, and how to make safe fixes without broadening the PR unnecessarily.

A reviewer pack is not a script. It is durable operating context for an AI coding agent.

## Pack Layout

Use this structure:

```text
packs/<reviewer-id>/
  README.md
  knowledge/
    context-loading.md
packs/shared/
  reviewer-runtime.md
```

Additional knowledge files are fine when they serve a distinct purpose. A pack may add a domain-specific checklist, or it may point to a bundled or target-repository skill that owns the checklist. Shared runtime, finding, handoff, and output instructions belong in `packs/shared/reviewer-runtime.md`. Keep the entrypoint short enough that a reviewer can quickly understand its job.

## Entrypoint README

The pack README should answer:

- what this reviewer is responsible for
- which files or systems it should inspect first
- what issues are high priority
- when it should edit code directly
- when it should stop and report an unresolved finding
- which knowledge files to read
- that shared runtime and output behavior comes from `packs/shared/reviewer-runtime.md`

Prefer domain-specific language. A Temporal reviewer should talk about workflow determinism, activity side effects, replay, retries, signals, and reconciliation. A Convex reviewer should route the agent to Convex guidance and skills in the target repository instead of duplicating Convex framework rules here.

## Context Loading

Context-loading guidance should make the reviewer efficient. List the smallest useful reading path first.

Good loading instructions:

- start with the touched domain files
- read generated or canonical guidance when the domain requires it
- inspect callers only when contract impact needs validation
- load specialized skills or references only when the change matches their purpose

Avoid telling every reviewer to read the entire repository or all docs before doing any work.

## Review Checklist

The checklist should capture domain hazards that a general-purpose reviewer may miss.

When a pack delegates domain review to a skill, prefer referencing that skill instead of duplicating its checklist in the pack.

Write checks as concrete failure modes:

- "workflow code calls `Date.now()` directly"
- "query filters after a wide table scan instead of using an index"
- "schema adds a required field without a migration path"

Avoid vague items like "make sure the code is clean" or "review for best practices".

## Scope

Specialized reviewers use soft scope guardrails. They should focus on matched files and their domain, but they may make cross-cutting edits when necessary for correctness.

Pack docs should say:

- which files are primary
- which adjacent files are worth reading to validate contracts
- which unrelated areas should usually be ignored

Do not try to enforce hard file restrictions in the pack. The orchestrator does not sandbox reviewer writes by file path.

## Fixing Versus Reporting

Reviewers are expected to fix concrete issues when the fix is clear and safe.

Use unresolved findings for:

- issues that are unsafe to change automatically
- issues that require product judgment
- issues outside the reviewer's domain
- follow-up work intended for a later reviewer or human

If the reviewer fixed an issue, it generally should not also report that issue as an unresolved finding. It can mention the fix in the summary and emit a handoff item when a downstream reviewer should adapt to it.

## Shared Runtime Instructions

`packs/shared/reviewer-runtime.md` is the coordination mechanism for behavior that should be identical across reviewers:

- fix-first expectations
- commit and push behavior
- unresolved finding shape
- handoff item shape
- structured result fields

Keep these rules out of individual pack READMEs unless a reviewer needs stricter domain-specific guidance.

## Handoff Items

Handoff items coordinate sequential reviewers.

Good handoff items are short:

- "Changed `convex/schema.ts` user fields; UI reviewer should verify profile rendering still matches the returned shape."
- "Adjusted workflow retry classification; later persistence reviewer should confirm run details still render the new status."

Poor handoff items are vague:

- "Check everything."
- "Made changes."
- "There might be issues."

Use `targetReviewerId` when a later reviewer is known. Use `null` when the handoff applies to any downstream reviewer.

## Tool Recommendations

`recommendedTools` in `reviewers.json` is advisory. It helps future runtimes and humans understand what tools are useful for the domain, but current reviewer execution is driven by the orchestrator runtime prompt and the markdown files.

Keep tool names generic and stable, such as `rg`, `pnpm`, `convex`, or `temporal`.

## Adding A Reviewer

1. Choose a stable `id`.
2. Create `packs/<id>/README.md`.
3. Add focused knowledge files under `packs/<id>/knowledge/`.
4. Add the reviewer to `reviewers.json`.
5. Make sure every path in `reviewers.json` exists.
6. Ensure the reviewer description is useful both to humans and runtime prompts.

The orchestrator will only run the reviewer when repo policy enables the reviewer ID and changed files match that policy's globs.

## Updating A Reviewer

When changing an existing reviewer:

- keep the reviewer ID stable
- preserve entrypoint paths unless there is a strong reason to move them
- update `knowledgeFiles` when files are added, removed, or renamed
- keep old guidance out if it conflicts with the orchestrator's current contract
- avoid embedding temporary incident context in evergreen docs

If a change alters how a reviewer should coordinate with other reviewers, update its handoff guidance.
