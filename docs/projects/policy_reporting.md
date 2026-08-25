# PolicyReporting

## Role

Cross-source service layer that combines MIPS and BriteCore data under a unified query/reporting API.

## What the project does

Based on the local `README.md`, `PolicyReporting` sits above source interfaces and DAL layers to provide a reporting-oriented service layer. It is designed to:

- fan out queries across multiple source systems
- assemble higher-level report objects
- support both CSV-backed and SQL-backed workflows
- serve as a foundation for future renderers, APIs, or UI layers

## Best-known documentation entry points

- `C:\PythonProjects\PolicyReporting\README.md`
- `C:\PythonProjects\PolicyReporting\CLAIMS_REPORTING_GUIDE.md`
- `C:\PythonProjects\PolicyReporting\QUICKSTART.md`
- `C:\PythonProjects\PolicyReporting\INSURANCE_REPORTING_ROADMAP.md`

## What it is strongest at

- cross-source reporting across MIPS and BriteCore
- service-layer query assembly
- SQL backend integration
- higher-level reporting workflows and future rendering/API layers

## Key local documentation themes

- stack positioning from raw parsers through service and rendering layers
- environment-variable based source configuration
- SQL Server backend configuration and probing
- SQL schema bootstrap and loader workflows

## Recommended use cases

Use `PolicyReporting` first when:

- you need one reporting interface across BriteCore and MIPS
- you want reporting workflows backed by SQL Server or mixed raw-data sources
- you are building higher-level report objects beyond a single source repo
- you need a service boundary for reporting APIs or rendered outputs

## Relationship to other projects

- can consume normalized logical-layer understanding from `britecore_mcp`
- can complement `britecore_sdk` for API-fed workflows
- can use exported/raw data patterns documented in `britecore-csv-loader`

