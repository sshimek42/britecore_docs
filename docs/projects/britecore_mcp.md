# britecore_mcp

## Role

Interactive MCP server and logical-layer discovery companion for BriteCore analysis.

## What the project does

Based on the local `README.md`, `britecore_mcp` is designed for read-only analysis workflows over BriteCore report data. It supports:

- local MCP server execution
- report discovery and runtime execution helpers
- cache rebuilding for BriteCore report snapshots
- smoke-testing realistic report requests
- prompt/tool orchestration for analysis workflows

## Best-known documentation entry points

- `C:\PythonProjects\BriteCore\britecore_mcp\README.md`
- `C:\PythonProjects\BriteCore\britecore_sql_reports\scripts\report_docs\report_tables_readme.md`
- `C:\PythonProjects\BriteCore\britecore_sql_reports\scripts\report_docs\report_table_doc_framework.md`
- `C:\PythonProjects\BriteCore\britecore_sql_reports\scripts\report_docs\report_join_starters.md`

## What it is strongest at

- report/view discovery
- logical SQL discovery via `v_logical_catalog`
- MCP-driven analysis workflows
- query-oriented guidance for the BriteCore reporting layer

## Key local documentation themes

- local server setup and credential configuration
- cache rebuild workflows via `rebuild_cache.py` and `rebuild_bc_report_cache.py`
- report-table documentation for `v_*` views
- practical join guidance for the logical layer

## Fast local run pattern

```powershell
cd C:\PythonProjects\BriteCore\britecore_mcp
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install .
```

Typical local server run:

```powershell
cd C:\PythonProjects\BriteCore\britecore_mcp
.\.venv\Scripts\python.exe -m britecore_mcp.server
```

## Cache and profile workflows

For logical-layer behavior that depends on cached artifacts, the local docs center around:

- `scripts\rebuild_cache.py` for cache refresh
- `scripts\rebuild_bc_report_cache.py` for report snapshot cache generation
- `britecore_sql_reports\scripts\config\bc_report_cache_profiles.json` for profile configuration

This is the main operational surface for preparing repeatable local report runs before container/image deployment.

## High-value report documentation assets

- `britecore_sql_reports\scripts\report_docs\report_tables_readme.md` (table-doc index)
- `britecore_sql_reports\scripts\report_docs\report_table_doc_framework.md` (authoring framework)
- `britecore_sql_reports\scripts\report_docs\report_join_starters.md` (query/join starter patterns)

Together, these provide a practical documentation workflow anchored on `v_logical_catalog`.

## Recommended use cases

Use `britecore_mcp` first when:

- you need to discover which report table or field answers a business question
- you are documenting or validating the logical SQL layer
- you want interactive assistance around report joins, grain, or field semantics
- you are building MCP-based internal analysis workflows

## Relationship to other projects

- pairs with `britecore_sdk` when API-side access or automation is needed
- pairs with `britecore_sql_reports` when SQL report definitions and report docs are needed
- pairs with `PolicyReporting` when cross-source reporting or SQL-backed service workflows are needed
- pairs with `britecore-csv-loader` when raw CSV relationship exploration is needed outside the logical SQL layer

## Where this fits in the reporting stack

Use `britecore_mcp` as the primary logical-layer documentation and exploration surface. Use other projects when you need:

- live API retrieval (`britecore_sdk`)
- SQL report definitions (`britecore_sql_reports`)
- raw export relationship reconstruction (`britecore-csv-loader`)
- unified multi-source service workflows (`PolicyReporting`)
