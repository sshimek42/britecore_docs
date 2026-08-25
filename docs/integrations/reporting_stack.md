# Reporting Stack Integration

This page explains how the current local BriteCore-related projects fit together in a reporting workflow.

## The four local layers

### 1) `britecore_sdk`
Use when you need live BriteCore API access.

Typical responsibilities:
- authenticate with API key or OAuth
- call BriteCore endpoints from Python
- build automations and API-backed integrations
- normalize API access behind reusable client wrappers

### 2) `britecore_mcp`
Use when you need interactive analysis against the BriteCore logical/reporting layer.

Typical responsibilities:
- guide report discovery with `v_logical_catalog`
- document `v_*` and `m_*` report objects
- support query-oriented workflows and MCP-assisted analysis
- provide practical join and grain guidance for report authors

### 3) `britecore-csv-loader`
Use when you need to work from raw exported CSVs rather than the logical SQL layer or API.

Typical responsibilities:
- load raw CSVs from export directories
- discover `*Id` relationships automatically
- assemble revision-centric and policy-centric snapshots
- inspect claim, item, policy, and contact linkages in raw exports

### 4) `PolicyReporting`
Use when you need a higher-level service layer across BriteCore and other systems such as MIPS.

Typical responsibilities:
- combine multiple source systems under one query API
- expose report-oriented query methods
- support SQL-backed and CSV-backed reporting workflows
- act as a future base for rendered reports or service endpoints

## Recommended decision tree

### If the question starts with "Can I call the BriteCore API to..."
Start with:
- `britecore_sdk`

### If the question starts with "What report table/view contains..."
Start with:
- `britecore_mcp`
- especially `v_logical_catalog`, report-table docs, and join guides

### If the question starts with "I only have raw CSV exports..."
Start with:
- `britecore-csv-loader`

### If the question starts with "I need a reusable reporting service across BriteCore and MIPS..."
Start with:
- `PolicyReporting`

## Example integrated workflow

### Scenario: build a claims reporting workflow

1. Use `britecore_mcp` docs to identify the logical report tables:
   - `v_claims`
   - `v_claims_dates`
   - `v_claims_contacts`
   - `v_claim_payments`
   - `v_claims_perils`

2. If raw CSV parity is needed, use `britecore-csv-loader` to trace the export-side relationship model:
   - `claims.csv`
   - `claim_items.csv`
   - `claim_payments.csv`
   - `claims_contacts.csv`
   - `claims_perils.csv`

3. If live system automation is needed, use `britecore_sdk` to retrieve or validate upstream operational data.

4. If the final deliverable must combine BriteCore with MIPS or another source, move the assembled logic into `PolicyReporting`.

## Important design principle

These projects should not compete to be the single source of truth for everything.

A cleaner split is:
- repo-specific implementation docs stay in the source repo
- cross-project architecture, workflow, and translation guidance live in `britecore_docs`

