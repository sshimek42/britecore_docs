# Consolidation Principles

This repo exists to reduce duplication and fill gaps, not to duplicate implementation details.

## Core rule

The authoritative home for implementation logic stays in the project that owns it. This repo is the explanation layer.

## What belongs here

Keep here only the information that is:

- cross-project and reusable
- difficult to find because it is split across multiple repos
- missing from the canonical implementation repos
- useful as a shared mental model for the ecosystem

Examples:

- repo ownership and responsibility mapping
- raw-to-warehouse-to-reporting flow
- lineage and grain guidance
- status and cancellation semantics
- repo-selection decisions by task
- common cross-project terminology

## What does not belong here

Do not duplicate the following unless there is a specific documentation need:

- raw SQL report definitions
- loader implementation internals
- API client implementation details
- MCP runtime code details
- CSV extractor logic in full
- project-specific operational runbooks that belong in the owning repo

## The value of this repo

This repo is most useful when it answers questions like:

- which repo owns this concern?
- what is the relationship between these systems?
- how does data move between raw, warehouse, and report layers?
- which project should I use for a task?
- what are the semantics of the shared BriteCore concepts I keep tripping over?

## Editorial standard

When adding new docs here, prefer:

- summaries over full implementation mirrors
- clear boundaries over broad linking
- task-oriented guidance over repository-by-repository duplication
- synthesized knowledge over copied text from source repos

## Keep the docs crisp

If a topic is already well covered in a canonical repo, link to that repo instead of reproducing it here.

The result should be a short, high-value guide to the ecosystem, not a second copy of every implementation asset.
