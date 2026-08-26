# Repo Selection Guide by Task

This page helps answer the most common question in the BriteCore stack: which repo should I use for this task?

## Quick chooser

### I need a lightweight source-level view

Use the layer that matches your access and desired amount of aggregation.

This is not a strict pipeline. You can start with:

- raw CSVs if you want point-in-time exports and offline access with minimal formatting
- the API if you want real-time or near-real-time structured domain objects
- the SQL/report layer if you want a preformatted, easy-to-query reporting view

### I need to inspect raw export structure

Use: `britecore-csv-loader`

Best for:

- exploring exported CSV datasets
- understanding `*Id` relationships
- reconstructing policy snapshots
- testing relationship assumptions before report design

### I need to land or normalize source data into warehouse tables

Use: `policy-data-warehouse`

Best for:

- raw-to-core-to-reporting workflows
- SQL-based validation
- staging landed data for reporting
- preserving raw semantics while creating reporting views

### I need a multi-source reporting answer across BriteCore and MIPS

Use: `PolicyReporting`

Best for:

- cross-source service-layer logic
- fan-out queries across data sources
- report object assembly
- shared reporting workflows across systems

### I need a NAIC-specific data extraction or report output

Use: `NAIC`

Best for:

- form mapping and normalization
- regulatory extraction logic
- report generation tasks
- validation of source readiness before a regulatory run

### I need to understand logical/report-table semantics

Use: `britecore_mcp` or `britecore_sql_reports`

Best for:

- logical-layer discovery
- report-table definitions
- field names and reporting grain
- report authoring and validation SQL

### I need API access or SDK-based automation

Use: `britecore_sdk`

Best for:

- live API call patterns
- auth/config usage
- client and integration automation

### I need ecosystem explanation, not implementation detail

Use: `britecore_docs`

Best for:

- architecture and edge-case summaries
- repo ownership and boundary rules
- semantic bridge knowledge
- onboarding and cross-project framing

## Decision rule

The simplest rule is this:

- source and implementation detail → owning repo
- ecosystem explanation and cross-project bridge → `britecore_docs`

If the task is ambiguous, start by asking:

1. Is this raw data, warehouse data, or reporting data?
2. Is the deliverable a source implementation change or a cross-project answer?
3. Is the output meant for regulators, SQL reporting, or operational analysis?

## Common confusion patterns

### “This looks like a report problem, but the raw CSVs are the only source I have.”

Check `britecore-csv-loader` first for raw structure and lineage. Then move to `policy-data-warehouse` or `PolicyReporting` when a reporting-ready shape is needed.

### “This is a status problem, not a report issue.”

Check the BriteCore docs for status and cancellation semantics; then decide whether the reporting layer needs normalized status or raw descriptive reason text.

### “I need the answer across sources.”

That is usually a `PolicyReporting` or warehouse-layer question, not a raw export question.

### “I need to produce a regulatory output.”

Likely `NAIC`, unless your source data still needs warehouse or raw validation first.

## Recommended default workflow

For many business and data-quality questions, the route is:

1. raw relationship check in `britecore-csv-loader`
2. warehouse semantics in `policy-data-warehouse`
3. matching/report assembly in `PolicyReporting`
4. regulatory output work in `NAIC`
5. ecosystem explanation in `britecore_docs`

This keeps the stack understandable without forcing every question into a single repo.
