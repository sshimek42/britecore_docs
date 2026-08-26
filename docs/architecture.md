# Architecture and Working Model

This page captures the current architecture and operating model for the BriteCore-facing documentation stack.

## Documentation purpose

`britecore_docs` is intentionally not a replacement for each source repo's own implementation docs. Instead, it acts as a shared translation layer:

- map the local ecosystem of repositories
- summarize the responsibilities of each project
- explain how they connect in reporting and data workflows
- keep cross-cutting guidance in one place
- reduce onboarding friction for analysts and engineers

## Layered view

```text
BriteCore source systems
        |
        +--> API layer: britecore_sdk
        +--> Logical discovery layer: britecore_mcp
        +--> SQL reporting layer: britecore_sql_reports
        +--> Raw export layer: britecore-csv-loader
        +--> Cross-source service layer: PolicyReporting
        |
        +--> britecore_docs (shared docs + cross-project conventions)
```

## Project boundaries

### `britecore_sdk`

This is the API-first integration layer.

Use it when the requirement is:

- live data retrieval from the BriteCore API
- automation around authenticated endpoints
- reusable Python client workflows
- configuration and auth management for runtime systems

### `britecore_mcp`

This is the logical reporting discovery layer.

Use it when the requirement is:

- report/view discovery
- field and view semantics
- `v_*` and `m_*` exploration
- join strategies and grain validation
- MCP-backed analyst guidance

### `britecore_sql_reports`

This is the SQL report definition and authoring layer.

Use it when the requirement is:

- report SQL definitions
- report-table documentation
- join starters and grain guidance
- report validation examples and reusable SQL assets

### `britecore-csv-loader`

This is the raw export reconstruction layer.

Use it when the requirement is:

- working from exported CSVs instead of a live API
- reconstructing `*Id` relationships across raw files
- validating policy or claim lineage from exported snapshots
- reverse-engineering grain and join assumptions from raw tables

### `PolicyReporting`

This is the cross-source service layer.

Use it when the requirement is:

- combining BriteCore data with MIPS or other upstream systems
- assembling high-level reporting objects
- creating a stable service boundary over multiple source systems
- supporting future rendered reporting or API deliverables

## Source-of-truth rule

The canonical source of implementation detail always belongs in the owning repository.

A good rule of thumb is:

- `britecore_docs` owns the shared map, explanation, and integration questions
- each project repo owns code, schema, and implementation details
- docs here summarize and connect, rather than duplicate everything blindly

## Recommended working cadence

For new work, follow this sequence:

1. Start from the business question.
2. Identify whether the answer should come from API, logical views, raw exports, or a multi-source reporting service.
3. Link to the canonical repo docs for implementation details.
4. Add a cross-project summary here when the pattern is reused or needs broader onboarding context.

## Typical cross-project flows

### Flow 1: Understand a report table

- start with `britecore_mcp`
- inspect `v_logical_catalog`
- follow report-table docs and join starters in `britecore_sql_reports`
- only move to downstream code or service layers when the data model is understood

### Flow 2: Validate against raw exports

- use `britecore-csv-loader` to reconstruct lineage from exported CSVs
- compare the raw relationships to the curated logical layer
- look for mismatches in grain, foreign keys, or revision semantics

### Flow 3: Build a live integration

- use `britecore_sdk` for authenticated API access
- keep the logic isolated from report-view semantics
- feed the normalized data into downstream reporting or service patterns

### Flow 4: Combine sources into one reporting product

- normalize the domain model in `PolicyReporting`
- use report-table knowledge from `britecore_sql_reports`
- use raw CSV relationship checks when export-based validation is needed

## Decision guide

If the question is about:

- API access -> `britecore_sdk`
- report/view semantics -> `britecore_mcp`
- SQL report definitions -> `britecore_sql_reports`
- raw export relationship tracing -> `britecore-csv-loader`
- multi-source reporting service -> `PolicyReporting`
- documentation coordination -> `britecore_docs`

## Documentation quality principles

Keep these principles consistent across the docs set:

- explain the role of the component before the implementation details
- link back to canonical source docs instead of copying them wholesale
- call out grain, join, and source-of-truth assumptions explicitly
- prefer decision-oriented content over raw product history
- keep examples concrete and scoped to a single analyst or engineering workflow

## Future growth areas

This docs hub is most valuable when it collects patterns that are reused across teams, such as:

- report-table discovery playbooks
- logical-layer review checklists
- raw-export validation recipes
- API + reporting handoff guidance
- onboarding walkthroughs for new analysts
