# britecore_sql_reports

## Role

This project is the source-of-truth for BriteCore SQL report definitions and report-authoring assets.

It owns the reporting artifacts that were intentionally separated from `britecore_mcp` so the MCP layer can stay focused on discovery, runtime execution, and the logical catalog.

## What the project does

It includes:

- report SQL definitions
- report-table documentation
- join starters and grain guidance
- SQL report validation notes

## Source-of-truth boundary

- `britecore_sql_reports` = canonical report SQL and report docs
- `britecore_mcp` = runtime discovery, cache files, and operational report-execution metadata
- `britecore_docs` = architecture and explanatory context across the ecosystem

Runtime cache/config artifacts are intentionally not treated as SQL report source-of-truth.

## Best-known documentation entry points

- `C:\PythonProjects\BriteCore\britecore_sql_reports\README.md`
- `C:\PythonProjects\BriteCore\britecore_sql_reports\agent.md`
- `C:\PythonProjects\BriteCore\britecore_sql_reports\scripts\report_docs\report_tables_readme.md`
- `C:\PythonProjects\BriteCore\britecore_sql_reports\scripts\report_docs\report_table_doc_framework.md`
- `C:\PythonProjects\BriteCore\britecore_sql_reports\scripts\report_docs\report_join_starters.md`

## What it is strongest at

- report SQL authoring
- reusable report definitions
- report-table documentation
- join and grain guidance for SQL reporting
- cache/profile metadata for report execution and validation

## Key local documentation themes

- report metadata organization
- one-file-per-report convention
- summary/detail sheet patterns
- required parameter handling for MCP-ready reports
- validation and grain checks before a report is treated as production-ready

## Relationship to other projects

- pairs with `britecore_mcp` for report discovery and runtime execution
- complements `britecore_sdk` when report metadata must be fetched from the live API
- sits above `britecore-csv-loader` when raw export validation is needed
- is explained by `britecore_docs` as part of the full BriteCore architecture map
