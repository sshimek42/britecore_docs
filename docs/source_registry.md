# Source Registry

This page records the current local source repositories and their canonical documentation entry points.

## Local source repositories

### 1) `britecore_mcp`
- Path: `C:\PythonProjects\BriteCore\britecore_mcp`
- Main entry point: `README.md`
- Canonical role: MCP server behavior, logical-layer discovery, runtime report execution helpers

### 2) `britecore_sql_reports`
- Path: `C:\PythonProjects\BriteCore\britecore_sql_reports`
- Main entry point: `README.md`
- Reporting docs index: `scripts/report_docs/report_tables_readme.md`
- Report doc framework: `scripts/report_docs/report_table_doc_framework.md`
- Report join guide: `scripts/report_docs/report_join_starters.md`
- Cache profile config: `scripts/config/bc_report_cache_profiles.json`
- Canonical role: SQL report definitions, report-writing guidance, and report documentation assets

### 3) `britecore_sdk`
- Path: `C:\PythonProjects\BriteCore\britecore_sdk`
- Main entry point: `README.md`
- Additional major docs:
  - `GETTING_STARTED.md`
  - `ARCHITECTURE.md`
  - `API.md`
  - `CONFIG_MANAGEMENT.md`
- Canonical role: SDK installation, API usage, auth/config patterns, client architecture

### 4) `policy-data-warehouse`
- Path: `C:\PythonProjects\data\policy-data-warehouse`
- Main entry point: `README.md`
- Additional major docs:
  - `data/docs/BRITECORE_DATA_STRUCTURE.md`
  - `docs/project_notes/WORKFLOW_GUIDE.md`
- Canonical role: warehouse landing layer, raw/core/reporting schema boundaries, SQL-backed cross-source reporting, BriteCore status/cancellation semantics

### 5) `NAIC`
- Path: `C:\PythonProjects\data\NAIC`
- Main entry point: `README.md`
- Additional major docs:
  - `docs/BRITECORE_INTEGRATION_GUIDE.md`
  - `docs/BRITECORE_DATA_MAPPING.md`
  - `docs/BRITECORE_TO_NAIC_SOURCE_JOIN_DIAGRAM.md`
- Canonical role: regulatory extraction, BriteCore-to-NAIC mapping, config-driven report generation, validation before report delivery

### 6) `PolicyReporting`
- Path: `C:\PythonProjects\PolicyReporting`
- Main entry point: `README.md`
- Canonical role: cross-source reporting service layer, SQL-backed query workflows, reporting integration

### 7) `britecore-csv-loader`
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
