# Team Operating Manual

This document defines the practical operating model for the BriteCore documentation and reporting stack.

## Purpose

This project exists to make the local BriteCore ecosystem easier to navigate, not to replace the canonical implementations in the owning repositories.

The default rule is:

- code and implementation details stay in the repo that owns them
- cross-project coordination lives in `britecore_docs`
- shared reporting logic and patterns are documented here once they have stabilized

## Primary responsibilities by repo

### `britecore_docs`

`britecore_docs` is the synthesis layer.

It is responsible for:

- architecture summaries
- operating guidance
- source registry and repo handoff points
- reporting mental models
- cross-project onboarding and quickstarts
- curated references back to the canonical implementation repos

### `britecore_mcp`

Primary responsibility:

- report discovery
- logical catalog use
- `v_*` and `m_*` reporting guidance
- join starters and analyst tooling

### `britecore_sdk`

Primary responsibility:

- API access and automation
- auth/configuration patterns
- reusable Python client logic
- integration hooks for live BriteCore data retrieval

### `britecore-csv-loader`

Primary responsibility:

- raw CSV relationship discovery
- export lineage reconstruction
- revision and policy snapshot relationships
- validation of raw relationship assumptions

### `PolicyReporting`

Primary responsibility:

- cross-source service layer
- unified query/report assembly across sources
- SQL or mixed-source reporting orchestration
- final service-facing reporting abstractions

## Source-of-truth rule

Every document should make one of these claims clearly:

- this is authoritative in the owning repo
- this is a curated summary in `britecore_docs`
- this is a cross-project pattern worth standardizing

When in doubt, prefer links back to the original file over copying the full implementation detail.

## Workflow standard

### Start with the question

Use this order when working on a reporting or integration problem:

1. Define the business question
2. Define the grain and required output shape
3. Identify the likely owner repo
4. Locate the canonical docs in the owning repo
5. Summarize or standardize only what is cross-cutting

### Validate the assumptions

Before finalizing a reporting pattern, verify:

- the anchor object is correct
- the grain is stable
- joins are necessary and not multiplying rows
- the report matches the underlying source semantics
- the raw CSV or SDK source has been checked when needed

### Document the ownership

Every decision should document:

- what repo owns the logic
- what repo owns the canonical docs
- which repo is the operational reference for the workflow
- what patterns were standardized in `britecore_docs`

## Reporting triage decision tree

### If the ask is about semantics or table discovery

Start with `britecore_mcp`.

Typical questions:

- What table has this field?
- Which logical layer should I use?
- What is the right grain for this report?

### If the ask is about API access

Start with `britecore_sdk`.

Typical questions:

- Can I pull this from the live API?
- How do I authenticate?
- What client pattern should I use?

### If the ask is about raw CSV lineage

Start with `britecore-csv-loader`.

Typical questions:

- What is the relationship pattern behind these exports?
- How should I reconstruct this raw lineage?
- Is the logical layer matching the export semantics?

### If the ask is about a multi-source service boundary

Start with `PolicyReporting`.

Typical questions:

- How do I unify BriteCore data with other sources?
- What service contract does this reporting flow need?
- What is the final object model for downstream use?

## Documentation quality bar

A page is ready for shared use when it does all of the following:

- names the owning repo and canonical source
- explains what the component is for
- explains when to use it and when not to use it
- uses clear example paths and commands
- calls out grain, source assumptions, and join caveats if relevant
- avoids copying long implementation details that belong elsewhere

## Change-management expectations

### When updating a repo-specific doc

Update the source repo directly.

### When updating shared guidance

Update `britecore_docs` and keep a note if the summary reflects a source repo change.

### When a pattern is repeated across multiple repos

Promote it to a shared guidance page once it becomes common enough to matter beyond a single workflow.

## Review cadence

Use a lightweight review rhythm:

- source docs reviewed when the repo changes
- shared docs reviewed when cross-project patterns materially change
- key reporting guides reviewed when new business workflows are introduced

## Core team principles

1. Start with the business question.
2. Do not confuse repo-specific implementation docs with shared architecture docs.
3. Prefer canonical ownership over duplication.
4. Validate grain before finalizing report joins.
5. Keep the docs practical, decision-oriented, and link-heavy.

## Quick references

- [Architecture](architecture.md)
- [Source Registry](source_registry.md)
- [Project Map](project_map.md)
- [Onboarding](onboarding.md)
- [Quickstarts](quickstarts.md)
- [Reporting Playbook](reporting/reporting_playbook.md)
