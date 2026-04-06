# Reviewer Packs

This repository contains specialized reviewer packs for the PR review orchestrator.

## Purpose

Reviewer packs let the orchestrator load deep, domain-specific knowledge only when a pull request needs it.

This avoids bloating the context of general-purpose agents and gives specialized reviewers a clear, focused area of responsibility.

## Layout

- `reviewers.json`
  - machine-readable registry of available reviewers
- `packs/<reviewer-id>/README.md`
  - entrypoint for that reviewer pack
- `packs/<reviewer-id>/knowledge/*.md`
  - supporting knowledge files for that reviewer

## Consumption Contract

The orchestrator should:

1. resolve a reviewer ID from `reviewers.json`
2. provide the reviewer pack path to the agent
3. instruct the agent to start with the pack `README.md`
4. let the agent load additional markdown files from `knowledge/` as needed
5. record the current git commit SHA of this repository in the run details

## Notes

- This repository is the source of truth for reviewer definitions.
- Repo policy in the orchestrator should only enable reviewer IDs and define their order.
- Reviewers may recommend CLIs, local docs, or optional MCP capabilities, but knowledge should primarily be loaded from markdown in this repository.
