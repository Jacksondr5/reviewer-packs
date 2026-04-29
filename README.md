# Reviewer Packs

This repository contains specialized reviewer packs for the PR review orchestrator. A reviewer pack is durable markdown knowledge that an AI coding agent loads when the orchestrator asks it to perform a focused review pass on a pull request.

## Purpose

Reviewer packs let the orchestrator load deep, domain-specific knowledge only when a pull request needs it.

This avoids bloating the context of general-purpose agents and gives specialized reviewers a clear, focused area of responsibility.

Specialized reviewers are fixer-reviewers. They should fix concrete issues when the fix is clear and safe, report only unresolved or follow-up issues, and emit handoff items when later reviewers should account for their work.

## System Context

The orchestrator lives in a separate repository. It polls GitHub, runs one Temporal workflow per PR, prepares a PR workspace, launches agents, publishes committed changes, and persists run history in Convex.

The workflow runs actions in this order:

1. resolve merge conflicts when GitHub reports the PR is conflicted
2. fix allowlisted failing status checks
3. handle unresolved Code Rabbit review items
4. run matching specialized reviewers one at a time

This repository owns the reviewer knowledge loaded during step 4. It does not own workflow orchestration, repo policy, workspace setup, or persistence.

## Layout

- `reviewers.json`
  - machine-readable registry of available reviewers
- `packs/<reviewer-id>/README.md`
  - entrypoint for that reviewer pack
- `packs/<reviewer-id>/knowledge/*.md`
  - supporting knowledge files for that reviewer
- `docs/reviewer-authoring.md`
  - guidance for adding or updating reviewer packs
- `docs/orchestrator-contract.md`
  - the contract between this repository and the orchestrator runtime

## Consumption Contract

The orchestrator should:

1. resolve a reviewer ID from `reviewers.json`
2. provide the reviewer pack path to the agent
3. instruct the agent to start with the pack `README.md`
4. let the agent load additional markdown files from `knowledge/` as needed
5. record the current git commit SHA of this repository in the run details

At runtime the agent also receives PR metadata, changed files, matched files, prior reviewer summaries, and later reviewer information from the orchestrator.

## Registry Entries

Each entry in `reviewers.json` has:

- `id`: stable machine ID used by repo policy
- `title`: human-readable reviewer name
- `description`: concise purpose statement
- `entrypoint`: pack README path
- `knowledgeFiles`: extra markdown paths exposed to the reviewer
- `recommendedTools`: advisory tool names that may help the reviewer

Keep reviewer IDs stable. The orchestrator uses them to connect repo policy, run history, and pack metadata.

## Authoring Principles

- Keep guidance evergreen and domain-specific.
- Prefer concrete failure modes over generic code-review advice.
- Keep context-loading instructions efficient.
- Make scope boundaries clear without forbidding necessary cross-cutting fixes.
- Use handoff guidance to coordinate sequential reviewers.
- Avoid storing secrets, deployment-specific paths, or temporary incident notes in pack docs.

## Notes

- This repository is the source of truth for reviewer definitions.
- Repo policy in the orchestrator should only enable reviewer IDs and define their order.
- Reviewers may recommend CLIs, local docs, or optional MCP capabilities, but knowledge should primarily be loaded from markdown in this repository.
- Current orchestrator policy stores a `promptId`, but specialized reviewer runtime context currently comes from this repository's entrypoints and knowledge files plus the orchestrator's runtime prompt.
