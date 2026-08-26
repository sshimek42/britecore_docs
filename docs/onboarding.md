# Onboarding Guide

This guide helps new analysts and engineers get productive quickly across the local BriteCore documentation and tooling stack.

## What is in scope

The local ecosystem generally breaks into four roles:

- `britecore_sdk` — API-first access layer
- `britecore_mcp` — report discovery and logical-layer analysis
- `britecore-csv-loader` — raw export relationship reconstruction
- `PolicyReporting` — cross-source reporting service layer

`britecore_docs` sits above them as a coordination and translation layer.

## Recommended workspace layout

Keep these folders in the same IDE workspace:

- `C:\PythonProjects\BriteCore\britecore_docs`
- `C:\PythonProjects\BriteCore\britecore_mcp`
- `C:\PythonProjects\BriteCore\britecore_sdk`
- `C:\PythonProjects\PolicyReporting`
- `C:\PythonProjects\data\britecore-csv-loader`

This makes it easier to move between:

- source-of-truth docs
- report-table discovery pages
- SDK examples
- raw CSV lineage work
- cross-source reporting service logic

## First decision: what kind of problem is this?

### API access is required

Use `britecore_sdk` when the problem is about:

- authenticated HTTP calls
- client wrappers
- automation and integration logic
- configuration-driven access patterns

### Reporting semantics are required

Use `britecore_mcp` when the problem is about:

- which report view contains the needed data
- field semantics and descriptions
- logical catalog exploration
- join patterns and grain assumptions

### Raw export relationships are required

Use `britecore-csv-loader` when the problem is about:

- CSV-based lineage reconstruction
- `*Id` relationship discovery
- revision-centric or policy-centric snapshots
- comparing raw export structure to the logical layer

### A reusable reporting service is required

Use `PolicyReporting` when the problem is about:

- cross-source query assembly
- shared report object contracts
- SQL-backed or mixed-source reporting flows
- higher-level service boundaries beyond a single repo

## Day 1 workflow

### Step 1: Understand the business question

Before writing SQL or code, clarify:

- what metric or fact is needed
- what grain the output should have
- what key dimensions define the result
- what source system owns the canonical record

### Step 2: Identify the anchor object

Good anchors are often:

- policy or in-force record
- claim or claim item
- premium transaction
- contact or insured entity
- revision or audit record

Start from the object that matches the business question, not from the largest table in the warehouse.

### Step 3: Discover the right reporting surface

Use the local docs and catalog sources to answer:

- which view contains the fact tables
- which view adds descriptive dimensions
- which joins are required to answer the business question

The best starting point is usually `v_logical_catalog` in `britecore_mcp`.

### Step 4: Validate raw data assumptions

If the report is complex, unstable, or built from exports, validate the relationship assumptions in `britecore-csv-loader`.

This is especially important when:

- joins are exploding unexpectedly
- row counts do not match the expected grain
- the report depends on revisions, claims, or policy snapshots

### Step 5: Implement and document

As the solution stabilizes, document:

- the business rule
- the anchor table
- the key joins
- the row grain
- where the canonical logic lives
- which repo owns the implementation details

## Recommended learning path

### For analysts

1. Read `docs/reporting/logical_layer.md`
2. Review `docs/reporting/join_starters.md`
3. Explore `britecore_mcp` report tables and `v_logical_catalog`
4. Use `britecore-csv-loader` only when validating raw export assumptions
5. Keep SQL/report logic documented in the source repo or a shared reporting guide

### For engineers

1. Review `docs/architecture.md`
2. Read the project-specific overview pages in `docs/projects/`
3. Use `britecore_sdk` for API automation and integration patterns
4. Use `PolicyReporting` when a multi-source service layer is warranted
5. Keep repo-specific implementation details in the owning repository

## Common pitfalls

- Starting from the biggest table instead of the right business object
- Joining too many enrichment views before defining grain
- Mixing raw export and logical-report semantics without a validation step
- Copying docs into `britecore_docs` without a source note and review cadence
- Treating `britecore_docs` like a canonical implementation repo instead of a curated layer

## Standard operating principles

Follow these four principles throughout the stack:

1. Anchor the answer in the business question.
2. Respect the source-of-truth boundary.
3. Validate grain before finalizing joins.
4. Document the repo where the implementation lives.

## Helpful first pages

- [Architecture](architecture.md)
- [Project Map](project_map.md)
- [Source Registry](source_registry.md)
- [Reporting Logical Layer](reporting/logical_layer.md)
- [Reporting Playbook](reporting/reporting_playbook.md)
- [Quickstarts](quickstarts.md)
