# Reviewer Pack Agent Guide

This repository contains specialized reviewer packs for the PR review orchestrator. Agents working here are usually improving the knowledge and operating instructions used by later code-review agents, not changing the orchestrator itself.

## Product Context

The orchestrator is an AI-driven pull request review system built around one long-running Temporal workflow per GitHub pull request. A local poller wakes the workflow when PR state changes. The workflow fetches fresh GitHub state, chooses one next action, runs at most one code-changing agent at a time, and records summaries, artifacts, and errors in Convex.

Specialized reviewers run late in the workflow:

1. merge conflicts are resolved first when GitHub reports the PR is conflicted
2. allowlisted failing checks are handled
3. unresolved Code Rabbit review items are handled
4. matching specialized reviewers run one at a time

Reviewer packs exist so those specialized reviewers can load deep domain knowledge only when a PR needs it.

## What This Repo Owns

This repo is the source of truth for:

- `reviewers.json`, the machine-readable reviewer registry
- each reviewer pack entrypoint at `packs/<reviewer-id>/README.md`
- supporting markdown knowledge files under `packs/<reviewer-id>/knowledge/`
- optional recommendations for tools a reviewer may use

The orchestrator repo owns:

- polling GitHub
- Temporal workflow state and action ordering
- repo policy and reviewer enablement
- matching changed files to reviewer IDs
- preparing PR workspaces
- launching the agent runtime
- publishing committed changes
- persisting run history to Convex

Keep that boundary clear. Reviewer packs should describe how a reviewer thinks and works; they should not duplicate orchestration logic.

## Registry Contract

Every reviewer used by the orchestrator must have an entry in `reviewers.json`:

- `id`: stable machine ID, also used by repo policy
- `title`: human-readable name
- `description`: concise purpose statement
- `entrypoint`: path to the reviewer pack README
- `knowledgeFiles`: additional markdown files the orchestrator exposes to the reviewer
- `recommendedTools`: optional tool names that may help the reviewer

Use stable IDs. Renaming an ID can break repo policies and run-history interpretation.

## Reviewer Pack Contract

Each pack should have:

- a clear primary responsibility
- context-loading instructions
- scope boundaries
- a review checklist, or a domain skill that provides the checklist
- guidance for when to edit code directly
- guidance for unresolved findings
- handoff guidance for later reviewers, usually through `packs/shared/reviewer-runtime.md`

Reviewer instructions should be evergreen. Avoid dates, deployment-specific paths, personal secrets, current incident details, or assumptions that only hold for one repository.

## Runtime Expectations

At runtime, a specialized reviewer receives:

- repository, PR number, branch, head SHA, title, and body
- the full changed-file list
- the subset of files that matched this reviewer
- the reviewer pack path
- the entrypoint path
- the list of additional knowledge files
- summaries and handoff items from prior reviewers
- the reviewers that will run later

The reviewer is expected to start with the pack entrypoint, load relevant knowledge files, inspect the target PR workspace, fix concrete issues when safe, and return structured output.

## Fix-First Behavior

Specialized reviewers are fixer-reviewers. They may edit code when the fix is clear and within their domain.

Reviewer pack guidance should reinforce these rules:

- focus primarily on the reviewer's domain and matched files
- allow cross-cutting edits when required for correctness
- prefer fixing concrete issues over only reporting them
- leave findings only for unresolved or follow-up issues
- emit handoff items when later reviewers should account for a change
- do not tell the reviewer to push; the orchestrator verifies and publishes committed changes

## Coordination Model

Reviewers do not share an agent session. Coordination happens through workflow-provided summaries and handoff items.

Write handoff guidance so it is short and operational:

- what changed
- what a later reviewer should verify
- which interface, file, or behavior is affected

Do not rely on hidden state, side channels, or assumptions that another reviewer read the same local scratch notes.

## Result Shape Awareness

The orchestrator expects specialized reviewers to return structured fields including:

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

Pack docs do not need to restate the full schema every time, but reviewer instructions should be compatible with this result shape.

## Current Caveats

- Repo policy stores a `promptId`, but the current orchestrator runtime does not use it. Active reviewer context comes from this repo's entrypoint and knowledge files plus the orchestrator's runtime prompt.
- Reviewer selection is currently based on repo policy file globs and existing `reviewerRuns`, not the registry's `recommendedTools`.
- Specialized reviewers currently run sequentially. Do not design pack behavior around concurrent reviewer execution.
- The orchestrator records summaries and artifacts. Avoid instructing reviewers to preserve large raw code excerpts in output.

## Authoring Standards

Use concise markdown. Prefer specific review heuristics and failure modes over generic advice.

Good reviewer guidance:

- names concrete files or APIs to inspect
- distinguishes high-impact correctness issues from style preferences
- says when not to act automatically
- explains what evidence should support a finding
- gives handoff examples for downstream reviewers

Weak reviewer guidance:

- repeats broad coding principles
- requires reading unrelated files on every run
- tells the reviewer to make large opportunistic refactors
- assumes a single target repository layout unless the reviewer is explicitly repo-specific
- duplicates orchestrator implementation details that belong in the orchestrator repo

## Domain Skill Guidance

This repository should not include generated AI guidance or domain skill copies for reviewers. Convex and Temporal reviewer packs must defer to domain guidance and skills available in the target PR repository or reviewer workspace.
