# britecore_mcp

## Role

Interactive MCP server and logical-layer reporting companion for BriteCore analysis.

## What the project does

Based on the local `README.md`, `britecore_mcp` is designed for read-only analysis workflows over BriteCore report data. It supports:

- local MCP server execution
- report discovery and documentation
- cache rebuilding for BriteCore report snapshots
- smoke-testing realistic report requests
- prompt/tool orchestration for analysis workflows

## Best-known documentation entry points

- `C:\PythonProjects\BriteCore\britecore_mcp\README.md`
- `C:\PythonProjects\BriteCore\britecore_mcp\scripts\report_tables_readme.md`
- `C:\PythonProjects\BriteCore\britecore_mcp\scripts\report_table_doc_framework.md`
- `C:\PythonProjects\BriteCore\britecore_mcp\scripts\report_join_starters.md`

## What it is strongest at

- report-table documentation
- logical SQL discovery via `v_logical_catalog`
- MCP-driven analysis workflows
- query-oriented guidance for the BriteCore reporting layer

## Key local documentation themes

- local server setup and credential configuration
- cache rebuild workflows via `rebuild_cache.py` and `rebuild_bc_report_cache.py`
- report-table documentation for `v_*` views
- practical join guidance for the logical layer

## Recommended use cases

Use `britecore_mcp` first when:

- you need to discover which report table or field answers a business question
- you are documenting or validating the logical SQL layer
- you want interactive assistance around report joins, grain, or field semantics
- you are building MCP-based internal analysis workflows

## Relationship to other projects

- pairs with `britecore_sdk` when API-side access or automation is needed
- pairs with `PolicyReporting` when cross-source reporting or SQL-backed service workflows are needed
- pairs with `britecore-csv-loader` when raw CSV relationship exploration is needed outside the logical SQL layer

