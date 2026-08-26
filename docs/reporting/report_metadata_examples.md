# Report Metadata and API Examples

This page captures the current SQL report metadata shape from the MCP/report catalog workflow and the temp report files.

The report SQL definitions now live in `britecore_sql_reports`; `britecore_mcp` remains the discovery and runtime companion.

## Validation summary

I checked the current temp SQL report set and all files follow the same basic structure:

- 76 reports total
- 76 use `sql_runner`
- 58 export to Excel
- 18 export to CSV
- all include `Report Info` and `Report Definition`
- all include at least one SQL sheet

That means the temp folder is a reliable current catalog source for report metadata examples.

## What the MCP-side metadata looks like

The report payload consistently includes:

- `id`
- `category_id`
- `name`
- `description`
- `output_format`
- `delivery_type`
- `date_type`
- `report_parameters`
- `report_runner`
- `required_memory`
- `output_filename`
- `report_sheets`

Each sheet then adds:

- `id`
- `name`
- `order`
- `formatting`
- `sql_query`
- `include_totals_row`

## Representative examples

| Report | Format | Date Type | Parameters | Sheets | Notes |
|---|---|---|---|---|---|
| Accounting Detail Report | Excel | Range | `StartDate`, `EndDate` | 2 | Classic multi-sheet summary/detail report |
| Claims by Loss Date (MCP Ready) | CSV | Range | `StartDate`, `EndDate` | 1 | Custom extract style report with live date filtering |
| PIF2 | Excel | Single | `AsOfDate` | 11 | Large multi-sheet workbook with property/item JSON work |
| User to Role and Permissions | Excel | Single | None | 4 | Admin/reporting workbook with registry, summary, and rules tabs |

## What to look for in a report definition

### 1) Parameter shape

The most common parameter patterns are:

- `StartDate` / `EndDate` for date windows
- `AsOfDate` for point-in-time snapshots
- no parameters for static reference reports

### 2) Output shape

Two common patterns appear in the current catalog:

- Excel workbook reports with multiple sheets
- CSV extracts that behave like single-sheet analytical extracts

### 3) Sheet design

Sheets usually carry one of these roles:

- summary
- detail
- validation / scratch / working sheet
- registry or lookup sheet

That is why a report file should generally stay as one file per report: the report metadata and all sheet SQL stay together in a single readable package.

## MCP/API usage pattern

The basic flow from MCP or SDK is:

1. list the current SQL reports
2. fetch a single report definition by `report_id`
3. read the metadata and sheet SQL
4. compare the definition to the temp report file for validation or documentation

That maps directly to the examples already documented in `britecore_mcp`:

- `retrieve_sql_reports()`
- `retrieve_report(report_id=...)`

## Practical documentation rule

When documenting a report:

- keep the metadata and sheet SQL in the same report-specific page
- show the report parameters at the top
- call out whether the report is a snapshot, a date range extract, or a static reference workbook
- summarize the sheet purpose instead of duplicating every column twice

## Execution behavior examples

The temp report files alone show the definition shape, but live MCP execution adds an important extra layer: some reports have required runtime filters that are not obvious from the report name alone.

### Policy Type Catalog (MCP Ready)

Observed execution pattern:

```python
report_run = run_report(
    report_name="Policy Type Catalog (MCP Ready)",
    additional_report_configuration={
        "parameters": [
            {"name": "LineNameTerm", "value": "homeowner"}
        ]
    },
    end_date="2026-07-31"
)
```

This returned:

- `No data returned - Error - Missing required parameters: State`

When `State` was added:

```python
report_run = run_report(
    report_name="Policy Type Catalog (MCP Ready)",
    additional_report_configuration={
        "parameters": [
            {"name": "LineNameTerm", "value": "homeowner"},
            {"name": "State", "value": "WI"}
        ]
    },
    end_date="2026-07-31"
)
```

The report produced `Policy Type Catalog (MCP Ready).csv` in temp.

When `State=IL`, the report produced `Policy Type Catalog (MCP Ready)-IL.csv`, but that state had no data in BriteCore.

### Documentation implication

For MCP-ready reports, the docs should record:

- required parameters even when the report definition is not explicit enough
- whether a report returns no data for some state/line combinations
- whether the runtime injects a filename suffix or state-specific output name

That behavior belongs in the report-specific page, not just in the generic API examples page.
