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

## Fast local usage pattern

```python
from policy_reporting import CrossSourcePolicyQuery

query = CrossSourcePolicyQuery.from_env()
result = query.find_by_form("HO 002*")
print(result.summary())
```

This reflects the documented design goal: one query interface that can include BriteCore and MIPS sources when available.

## SQL backend workflow notes

From the local `README.md`, common operational patterns include:

- installing SQL extras (`.[sqlserver]`)
- probing connection health (`python -m policy_reporting.sql_connection --probe`)
- bootstrapping schema from `sql/sqlserver/*` folders
- selecting the SQL forms table via `POLICY_REPORTING_SQL_FORMS_TABLE`

For teams, these steps are the critical handoff between prototype reporting and productionized SQL-backed workflows.

## Source interface alignment

`PolicyReporting` is designed to sit above source interfaces and DAL layers. In practice, this makes it a good place to codify:

- cross-source normalization
- report object assembly
- stable service contracts for downstream API/UI layers

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

## Where this fits in the reporting stack

Use `PolicyReporting` when the requirement moves beyond single-repo analysis into a shared service layer. Keep table/field semantics anchored in `britecore_mcp` docs, then map those semantics into cross-source query services here.

