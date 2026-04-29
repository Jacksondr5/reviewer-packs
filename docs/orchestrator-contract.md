# Orchestrator Contract

This document describes how the PR review orchestrator consumes this repository.

## Ownership Boundary

The reviewer-pack repository owns reviewer knowledge. The orchestrator owns execution.

This repository provides:

- reviewer registry metadata
- pack entrypoints
- markdown knowledge files
- optional tool recommendations

The orchestrator provides:

- GitHub polling
- PR snapshot loading
- repo policy loading
- changed-file matching
- run de-duplication
- workspace preparation
- agent launch
- commit publication
- Convex persistence
- operator UI visibility

## Registry File

The orchestrator reads `reviewers.json` from the repository root.

Expected shape:

```json
{
  "reviewers": [
    {
      "id": "example-reviewer",
      "title": "Example Reviewer",
      "description": "Reviews a specific domain.",
      "entrypoint": "packs/example-reviewer/README.md",
      "knowledgeFiles": [
        "packs/example-reviewer/knowledge/review-checklist.md"
      ],
      "recommendedTools": ["rg"]
    }
  ]
}
```

All paths are relative to the reviewer-pack repository root. The current orchestrator resolves them into absolute paths before passing them to the reviewer agent.

## Policy And Selection

Repo policy in the orchestrator enables specialized reviewers separately from this registry. A policy reviewer definition includes:

- reviewer ID
- description
- file globs
- run policy
- prompt ID

The current runtime uses the policy ID to find the matching registry entry here. The policy array order defines execution order.

The orchestrator selects reviewers when changed files match the policy `fileGlobs`. It then filters prior successful runs:

- `once_per_pr`: skip if this reviewer already completed for the PR
- `once_per_sha`: skip if this reviewer already completed for the current head SHA

## Execution Order

Specialized reviewers run after earlier workflow actions are clear:

1. merge conflict resolution
2. fixable failing status checks
3. Code Rabbit review item handling
4. specialized reviewers

Reviewers run sequentially. If a reviewer creates and commits a change, the orchestrator observes the new head SHA, marks the workflow dirty, and starts a fresh reconciliation pass instead of continuing with stale PR state.

## Runtime Prompt Inputs

The orchestrator includes this reviewer-pack context in the specialized reviewer prompt:

- reviewer ID
- reviewer description from repo policy
- matched files
- reviewer-pack repository path
- reviewer-pack commit SHA
- entrypoint path
- additional knowledge file paths
- all changed files in the PR
- summaries and handoff items from prior reviewer runs
- later reviewers still expected in the sequence

Because the reviewer receives direct file paths, pack docs can instruct the reviewer to load files by path rather than embedding large content directly in prompts.

## Expected Reviewer Behavior

The runtime prompt asks reviewers to:

- focus primarily on their specialized domain and matched files
- make cross-cutting edits when required for correctness
- prefer fixing concrete issues directly
- create exactly one commit at the end if code was modified
- avoid pushing the branch themselves
- explain why no commit was created when no commit is made
- treat findings as unresolved or follow-up issues
- emit handoff items for later reviewers

Pack docs should align with those expectations.

## Structured Result

The specialized reviewer result is expected to include:

- `reviewerId`
- `matchedFiles`
- `overallSummary`
- `investigationSummary`
- `finalAssessment`
- `whyNoCommit`
- `commandsSummary`
- `didModifyCode`
- `didCommitCode`
- `findings`
- `handoffItems`

Each finding should include:

- `title`
- `actionSummary`
- `evidenceSummary`

Each handoff item should include:

- `targetReviewerId`, or `null`
- `summary`

## Persistence

The orchestrator records reviewer execution in Convex as both general PR run history and specialized reviewer run history. Stored details include reviewer ID, target head SHA, matched files, status, summary, structured result details, and reviewer-pack commit metadata.

The operator UI uses those records to answer what ran, why it ran, what changed, and what remains unresolved.

## Current Runtime Caveats

- `promptId` exists in repo policy but is not currently used by the specialized reviewer runtime.
- `recommendedTools` is registry metadata, not an execution guarantee.
- The orchestrator syncs or uses a local checkout of this repository before loading packs. If the checkout has local changes, the runtime may use those local changes as-is.
- The orchestrator does not enforce hard file-level write restrictions for specialized reviewers.
- Reviewer agents should not assume concurrent reviewer execution.

## Compatibility Guidance

To avoid breaking orchestrator consumption:

- keep `reviewers.json` valid JSON
- keep reviewer IDs stable
- keep registry paths accurate
- avoid removing a reviewer without first removing or updating any orchestrator repo policy that references it
- keep pack entrypoints useful as standalone starting points
- avoid relying on provider-specific global skills as the only source of reviewer knowledge
