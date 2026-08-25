# Project Map

## Roles by project

### `britecore_mcp`
Primary interactive analysis and report-surfacing layer.

Best for:
- MCP-based BriteCore analysis workflows
- report-table documentation
- logical SQL discovery via `v_logical_catalog`
- operational prompts and tool orchestration

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
        +--> britecore_mcp           (interactive logical-layer reporting)
        +--> britecore-csv-loader    (raw CSV linking and snapshots)
        +--> PolicyReporting         (cross-source reporting service layer)

britecore_docs sits above them as a documentation and integration layer.
```

## Suggested consolidated-doc categories

- Reporting model and logical layer
- Join patterns and grain guidance
- API and SDK usage
- Raw CSV relationship model
- Cross-source architecture
- Operational setup and troubleshooting

