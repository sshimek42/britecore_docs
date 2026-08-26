# britecore_docs

## Purpose

`britecore_docs` is the synthesis layer for the broader BriteCore ecosystem. Its job is to explain how the data stack fits together across repos, not to duplicate the raw implementation detail that lives in the owning projects.

This repo consolidates guidance across:

- `C:\PythonProjects\BriteCore\britecore_mcp`
- `C:\PythonProjects\BriteCore\britecore_sql_reports`
- `C:\PythonProjects\BriteCore\britecore_sdk`
- `C:\PythonProjects\PolicyReporting`
- `C:\PythonProjects\data\britecore-csv-loader`
- `C:\PythonProjects\data\policy-data-warehouse`
- `C:\PythonProjects\data\NAIC`

## What this project does not do

`britecore_docs` is not the canonical home for:

- report SQL definitions
- report cache profiles
- logical catalog implementation details
- raw export/loader code
- runtime MCP server code

Those assets remain owned by their source repos.

## Source-of-truth philosophy

The repo boundaries are deliberate:

- `britecore_mcp` is canonical for MCP runtime behavior, discovery tooling, and logical-layer interaction.
- `britecore_sql_reports` is canonical for SQL report definitions, report-table docs, validation SQL, and report metadata artifacts.
- `britecore_sdk` remains canonical for SDK configuration, API patterns, and automation flows.
- `PolicyReporting` remains canonical for cross-source reporting service logic and staging/reporting behavior.
- `britecore-csv-loader` remains canonical for raw CSV reconstruction and ingestion logic.
- `policy-data-warehouse` remains canonical for warehouse loading, raw/core/reporting schema boundaries, and cross-source SQL reporting plumbing.
- `NAIC` remains canonical for regulatory extraction and BriteCore-to-NAIC mapping, validation, and report generation.

`britecore_docs` is the curated map and explanation layer: architecture, onboarding, workflow guidance, and cross-project interpretation.

This repo intentionally focuses on the bridge knowledge that is often duplicated or missing across the stack: repo ownership, lineage, grain, task routing, and semantic conventions.

## Structure

```text
britecore_docs/
  README.md
  agent.md
  mkdocs.yml
  docs/
    index.md
    architecture.md
    onboarding.md
    quickstarts.md
    reporting/
      reporting_playbook.md
      sql_reporting_cookbook.md
      report_metadata_examples.md
    projects/
      britecore_mcp.md
      britecore_sql_reports.md
      britecore_sdk.md
      policy_reporting.md
      britecore_csv_loader.md
```

## Working rule

When a question is about the implementation detail, look in the owning project first. When a question is about how the ecosystem fits together, use `britecore_docs` as the explanatory layer.

This keeps the docs project useful without turning it into a second copy of the operational systems it describes.

## Canonical-source rule for centralized documentation

The docs hub should be a map, not a mirror. The owning project remains the canonical home for the details that change with the code.

- `britecore_sdk` is the canonical source for SDK API usage, installation, configuration, auth, examples, and troubleshooting.
- `britecore_mcp` remains the canonical source for MCP runtime behavior, discovery tooling, and logical-layer interaction.
- `britecore_sql_reports` remains canonical for report SQL and validation logic.
- `PolicyReporting` remains canonical for service-layer reporting behavior.
- `britecore-csv-loader` remains canonical for CSV reconstruction and ingestion logic.

For the SDK specifically, this project should link to the live project docs instead of duplicating them. Treat `C:\PythonProjects\BriteCore\britecore_sdk\README.md`, `GETTING_STARTED.md`, `CONFIG_MANAGEMENT.md`, `API.md`, `ARCHITECTURE.md`, and `TROUBLESHOOTING.md` as the authoritative documentation set.

