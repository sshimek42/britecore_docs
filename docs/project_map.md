# Project Map

## Roles by project

### `britecore_mcp`
Primary interactive analysis and logical-layer discovery layer.

Best for:
- MCP-based BriteCore analysis workflows
- logical SQL discovery via `v_logical_catalog`
- operational prompts and tool orchestration

### `britecore_sql_reports`
Primary SQL reporting definitions and authoring layer.

Best for:
- report SQL definitions
- report-table documentation
- join starter and grain guidance
- SQL report validation and examples

### `britecore_sdk`
Primary API access layer.

Best for:
- authenticated BriteCore API calls
- reusable Python integrations
- typed client code
- endpoint wrappers and config management

### `PolicyReporting`
Primary cross-source reporting service layer.

Best for:
- combining BriteCore and MIPS data
- SQL-backed or CSV-backed reporting workflows
- assembling higher-level report objects
- future output rendering and API-facing reporting services

### `policy-data-warehouse`
Primary warehouse and SQL reporting layer for landed BriteCore and MIPS data.

Best for:
- raw landing to `raw.*` tables
- `core.*` load and reporting view construction
- BriteCore status/cancellation semantics and cross-source normalization
- SQL-backed report validation and operational ingestion pipelines

### `NAIC`
Primary regulatory extraction and reporting layer for NAIC forms and cross-source coverage logic.

Best for:
- BriteCore-to-NAIC field mapping and normalization
- report generation for Part I/II/III/IV workflows
- validation checks before regulatory deliverables
- config-driven multi-company or multi-source runs

### `britecore-csv-loader`
Primary raw CSV relationship and ingestion layer.

Best for:
- loading raw CSV exports
- automatic relationship discovery across `*Id` fields
- revision-centric policy snapshots
- raw-CSV claims extraction workflows

## Cross-project architecture view

```text
BriteCore API / SQL / CSV exports
        |
        +--> britecore_sdk           (API access)
        +--> britecore_mcp           (interactive logical-layer discovery)
        +--> britecore_sql_reports   (SQL reporting definitions and authoring)
        +--> britecore-csv-loader    (raw CSV linking and snapshots)
        +--> policy-data-warehouse   (raw/core/reporting warehouse layer)
        +--> PolicyReporting         (cross-source reporting service layer)
        +--> NAIC                    (regulatory extraction and map-to-report layer)

britecore_docs sits above them as a documentation and integration layer.
```

## Suggested consolidated-doc categories

- Reporting model and logical layer
- SQL report definitions and authoring
- Join patterns and grain guidance
- API and SDK usage
- Raw CSV relationship model
- Cross-source architecture
- Operational setup and troubleshooting
