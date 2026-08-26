# Repo-by-Repo AI Usage Guide

This page is the practical playbook for using AI effectively inside each BriteCore project without blurring the boundary between implementation repos and the synthesis docs repo.

## Core rule

Use AI in the repo that owns the behavior.

- `britecore_sdk` → implementation and API behavior
- `britecore_sql_reports` → SQL/report definitions and report authoring
- `britecore_mcp` → logical discovery and MCP runtime behavior
- `britecore-csv-loader` → CSV extraction and raw relationship reconstruction
- `policy-data-warehouse` → warehouse landing and normalization logic
- `NAIC` → regulatory transformation and report generation
- `PolicyReporting` → service-layer orchestration and report assembly
- `britecore_docs` → cross-project synthesis, architecture, and bridge knowledge

## Recommended AI workflow by repo

### `britecore_sdk`

Use for:

- endpoint exploration
- auth and config troubleshooting
- API wrapper behavior questions
- implementing SDK helpers or wrapper logic
- testing live API workflows

Ask AI things like:

- “Find the SDK wrapper for claim retrieval and explain the required params.”
- “Which endpoint returns the policy ids and how do they chain into revision data?”
- “Explain the auth flow and site config precedence in this repo.”

Keep in mind:

- the SDK repo owns the canonical API semantics
- `britecore_docs` should summarize, not duplicate, this logic

### `britecore_sql_reports`

Use for:

- mapping logical-table fields to report SQL
- joining base views to bridge/detail views
- report metadata and table documentation
- testing SQL logic and validation queries

Ask AI things like:

- “Which report view is the best starting point for a claim status summary?”
- “What join chain connects revisions to claims and contacts?”
- “Explain the grain of `v_claims_contacts` vs `v_claims`.”

Keep in mind:

- report definitions live here
- use the docs repo to explain how it fits with CSV and API layers

### `britecore_mcp`

Use for:

- logical discovery and field introspection
- MCP workflow orchestration
- interpreting how report tables and metadata are surfaced
- building report discovery and validation flows

Ask AI things like:

- “Which report view should answer a policy lifecycle question?”
- “How should this table be discovered and validated through the MCP workflow?”
- “What is the best report metadata pattern for this request?”

Keep in mind:

- MCP is the logical discovery layer, not the raw source layer

### `britecore-csv-loader`

Use for:

- raw export reconstruction
- CSV-to-relationship reconstruction
- understanding how source tables split across files
- verifying raw field names and export grain

Ask AI things like:

- “Which CSV files are the master and bridge tables for claim relationships?”
- “How do I reconstruct a full contact record from the raw export?”
- “Which CSV file is the best match for the logical `v_contacts` view?”

Keep in mind:

- raw CSV exports are fragmented and split by concern
- use `britecore_docs` to explain the broader cross-layer mapping

### `policy-data-warehouse`

Use for:

- warehouse landing and normalization logic
- raw-to-core-to-report transformations
- validating report grain and source semantics
- issue analysis around warehouse table structure

Ask AI things like:

- “Which landing tables feed the report layer for revisions and claims?”
- “How do raw and normalized semantics differ in this warehouse model?”
- “Which staging tables should be used for a report reconstruction?”

Keep in mind:

- the warehouse is the normalization boundary, not the raw extraction layer

### `NAIC`

Use for:

- BriteCore-to-NAIC mapping and report generation
- regulatory export logic
- translation of source fields to required external layouts

Ask AI things like:

- “How does this BriteCore field map to the NAIC reporting schema?”
- “Which BriteCore source table is authoritative for this NAIC export?”
- “Where do the normalized values get transformed before report generation?”

Keep in mind:

- regulatory mapping is domain-specific and should stay in the NAIC project

### `PolicyReporting`

Use for:

- cross-source reporting orchestration
- service-level report assembly
- policy or claim reporting workflows that integrate multiple source layers

Ask AI things like:

- “Which data source should be the authoritative one for this service report?”
- “How do the report-service components combine SQL and source data?”
- “Which repo is the best owner for this piece of workflow logic?”

Keep in mind:

- this project is integration and reporting assembly, not the base source-of-truth repo

### `britecore_docs`

Use for:

- architecture and ecosystem synthesis
- bridge knowledge between repos
- repo ownership and source-of-truth guidance
- onboarding and workflow patterns
- how to reason across CSV, API, and SQL layers

Ask AI things like:

- “How do these repos fit together?”
- “What is the correct source-of-truth ownership for this task?”
- “How do I trace this record across CSV, API, and reporting layers?”

Keep in mind:

- this repo is intentionally not the implementation home for logic

## Prompt template for AI-assisted cross-project work

Use a prompt pattern like this:

```text
I’m working in <repo_name>.
Use this repo as the source of truth for implementation details.
Use britecore_docs only for ecosystem context, cross-project mapping, and decision support.
The task is: <task description>.
Please stay scoped to the owning repo unless the task explicitly needs cross-project bridge knowledge.
```

## Good default behavior for AI in this ecosystem

When working in any project:

- check the local repo docs first
- prefer the local implementation path over docs repo summaries
- use `britecore_docs` to answer “where does this fit” questions
- do not duplicate logic or implementation details into the docs repo
- keep the answer focused on the real owner of the behavior

## Final rule

If the work is about the actual behavior, code, or logic, it belongs in the owning repo.
If the work is about how the ecosystem connects, it belongs in `britecore_docs`.
