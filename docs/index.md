# britecore_docs

Welcome to the consolidated BriteCore documentation hub.

This project unifies the documentation and working model across the local BriteCore ecosystem:

- `britecore_mcp` — MCP server, logical-layer reporting, and report-table guidance
- `britecore_sql_reports` — SQL report definitions and report-authoring assets
- `britecore_sdk` — Python SDK for the BriteCore API
- `PolicyReporting` — cross-source reporting service layer
- `britecore-csv-loader` — raw CSV relationship discovery and extraction tooling
- `policy-data-warehouse` — landed warehouse model for raw/core/reporting data
- `NAIC` — regulatory extraction and BriteCore-to-NAIC mapping and reporting

## What this site is for

`britecore_docs` is the shared orientation layer for:

- architecture across tools
- data flow and integration patterns
- logical-layer reporting guidance
- cross-project setup and usage guides
- onboarding and reporting workflow standards
- curated references back to the source repos

## Start here

### If you need to answer a business question

- [Reporting Logical Layer](reporting/logical_layer.md) — how the reporting layer is modeled
- [Reporting Playbook](reporting/reporting_playbook.md) — how to define, validate, and deliver a report
- [SQL Reporting Cookbook](reporting/sql_reporting_cookbook.md) — practical SQL patterns and validation queries
- [Report Metadata Examples](reporting/report_metadata_examples.md) — current MCP-style report definitions and sheet patterns
- [Field Discovery Checklist](reporting/field_discovery_checklist.md) — the structured way to find the right fact and join path

### If you need to understand the ecosystem

- [Architecture](architecture.md) — how these repos fit together
- [Consolidation Principles](consolidation_principles.md) — the editorial rule for this repo
- [Source-of-Truth Matrix](source_of_truth_matrix.md) — which repo owns which concern
- [CSV / API / SQL Mapping](csv_api_sql_mapping.md) — how raw CSV names, API payloads, and SQL/report views align
- [Access Mode Decision Matrix](access_mode_decision_matrix.md) — when to use CSV, API, or SQL/report views
- [Bridge Table Guide](bridge_table_guide.md) — how relationship tables differ from base entity tables and why they multiply
- [Field Mapping Cheat Sheet](field_mapping_cheat_sheet.md) — side-by-side mappings for the main CSV/API/SQL entity shapes
- [Join Chains and Lineage](join_chains_and_lineage.md) — the repeatable join patterns that connect policies, claims, contacts, and properties
- [Query Quickstarts](query_quickstarts.md) — concrete SQL and API patterns for common business questions
- [Status and Enum Dictionary](status_and_enum_dictionary.md) — normalized meanings for active, deleted, terminated, claim state, policy state, and role flags
- [Relationship Catalog](relationship_catalog.md) — the major parent-child and bridge relationships across policies, claims, contacts, and properties
- [Domain Reference](domain_reference.md) — the domain-level map for claims, policy/revision records, contacts, properties, payments, and related objects
- [Claim Domain Guide](claim_domain_guide.md) — the practical lineage and join map for claim data
- [Policy and Revision Domain Guide](policy_domain_guide.md) — the practical lineage and join map for policy and revision data
- [Contact Domain Guide](contact_domain_guide.md) — how contact records connect to claims, revisions, and communication child tables
- [Property and Risk Domain Guide](property_domain_guide.md) — how property/risk data connects to revision and mortgagee records
- [Payment and Billing Domain Guide](payment_domain_guide.md) — how payment headers and item-detail data are linked across the platform
- [AI Workflow Guide for Other Repos](ai_workflow_guide.md) — how to work in the other BriteCore repos without duplicating implementation in the docs repo
- [Repo-by-Repo AI Usage Guide](repo_ai_usage_guide.md) — a practical guide for using AI in each project without losing repo boundaries
- [Reconciliation Checklist](reconciliation_checklist.md) — a repeatable validation flow for comparing raw CSV, API, and report-layer records
- [Common Gotchas and Anti-Patterns](common_gotchas_and_anti_patterns.md) — the recurring mistakes that make BriteCore joins and mappings look inconsistent
- [Record Trace Workflow](record_trace_workflow.md) — how to trace a policy, claim, or contact end to end across layers
- [Data Flow and Grain Guide](data_flow_and_grain_guide.md) — how raw data flows into reporting and why grain matters
- [BriteCore Status and Cancellation Semantics](britecore_status_and_cancellation_semantics.md) — how status and cancellation reasons differ across source systems
- [Lineage Guide](lineage_guide.md) — how revision, policy, claim, and property IDs connect
- [Repo Selection Guide](repo_selection_guide.md) — how to choose the right repo for a task
- [Help Center Insights](help_center_insights.md) — what the public BriteCore product documentation says about the platform structure
- [Data Storage and Handling](data_storage_and_handling.md) — how data moves through API, logical SQL/reporting, and CSV extracts
- [Data Lineage Reference](data_lineage_reference.md) — the end-to-end object lineage across policy, claim, revision, item, property, and contact data
- [Entity Relationship Matrix](entity_relationship_matrix.md) — the shared IDs and join chains that define the logical reporting graph
- [SQL Reporting Project](projects/britecore_sql_reports.md) — where report SQL and report docs now live
- [Project Map](project_map.md) — what each project is best at
- [Source Registry](source_registry.md) — canonical repos and entry points
- [Decision Matrix](decision_matrix.md) — choose the right repo for the task
- [Glossary](glossary.md) — shared terms, concepts, and reporting vocabulary
- [Operating Manual](operating_manual.md) — how the team works across the stack

### If you are onboarding or getting set up

- [Onboarding](onboarding.md) — the recommended first steps
- [Quickstarts](quickstarts.md) — repo-specific setup and usage patterns
- [Ingestion Workflow](ingestion_workflow.md) — how to add or refresh cross-project docs

## Common task flow

1. Clarify the business question and target grain.
2. Use `britecore_mcp` to find the logical reporting tables and fields.
3. Validate raw relationship assumptions with `britecore-csv-loader` when needed.
4. Use `britecore_sdk` for live API access when the task requires it.
5. Move cross-source reporting logic into `PolicyReporting` when a service layer is needed.
6. Document the final pattern here once it is stable enough to reuse.

## Recommended operating model

1. Keep detailed implementation docs in the source repos.
2. Use `britecore_docs` for shared architecture, onboarding, and cross-project guidance.
3. Link back to source files when the canonical implementation details live elsewhere.
4. Promote stable cross-repo patterns into this project once they become reusable.
5. Prefer decision-oriented documentation over raw duplication.

## Canonical ownership rules

- If the user needs the source-of-truth API or SDK guidance, start in `britecore_sdk`.
- If the user needs how the ecosystem fits together, use `britecore_docs`.
- If the user needs runtime or report-definition behavior, start in the owning project (`britecore_mcp`, `britecore_sql_reports`, etc.).

For the SDK in particular, the canonical documentation set is the SDK repo itself, and `britecore_docs` should summarize it rather than reproduce it.

## Current state

This hub includes:

- repo registry and architecture pages
- onboarding and quickstart guides
- reporting logical-layer references and cookbook guidance
- a dedicated SQL reporting project page
- a cross-project operating manual
- project summary pages for each local source repo
