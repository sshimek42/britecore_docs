# Source Registry

This page records the current local source repositories and their canonical documentation entry points.

## Local source repositories

### 1) `britecore_mcp`
- Path: `C:\PythonProjects\BriteCore\britecore_mcp`
- Main entry point: `README.md`
- Reporting docs index: `scripts/report_tables_readme.md`
- Report doc framework: `scripts/report_table_doc_framework.md`
- Report join guide: `scripts/report_join_starters.md`
- Canonical role: MCP server behavior, report-table docs, logical-layer reporting workflows

### 2) `britecore_sdk`
- Path: `C:\PythonProjects\BriteCore\britecore_sdk`
- Main entry point: `README.md`
- Additional major docs:
  - `GETTING_STARTED.md`
  - `ARCHITECTURE.md`
  - `API.md`
  - `CONFIG_MANAGEMENT.md`
- Canonical role: SDK installation, API usage, auth/config patterns, client architecture

### 3) `PolicyReporting`
- Path: `C:\PythonProjects\PolicyReporting`
- Main entry point: `README.md`
- Canonical role: cross-source reporting service layer, SQL-backed query workflows, reporting integration

### 4) `britecore-csv-loader`
- Path: `C:\PythonProjects\data\britecore-csv-loader`
- Main entry point: `README.md`
- Additional major docs:
  - `CSV_RELATIONSHIPS_OVERVIEW.md`
  - `docs/RELATIONSHIP_DIAGRAMS.md`
- Canonical role: raw CSV loading, automatic relationship discovery, revision/policy snapshot assembly, claims extractor workflows

## Recommended source-of-truth policy

- Prefer linking to original docs for repo-specific implementation details.
- Use `britecore_docs` for cross-repo summaries, shared mental models, and integration guides.
- When duplicating content into `britecore_docs`, record the originating file and review/update cadence.

