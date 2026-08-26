# Quickstarts

This page consolidates the most important setup and usage patterns across the local BriteCore ecosystem.

## Recommended order

Start here depending on the problem:

- Need live API access -> `britecore_sdk`
- Need report/view discovery -> `britecore_mcp`
- Need raw export relationship checks -> `britecore-csv-loader`
- Need a shared reporting service layer -> `PolicyReporting`

## `britecore_mcp`

### Local install

```powershell
cd C:\PythonProjects\BriteCore\britecore_mcp
python -m venv .venv
.\.venv\Scripts\Activate.ps1
python -m pip install .
```

### Typical run

```powershell
cd C:\PythonProjects\BriteCore\britecore_mcp
.\.venv\Scripts\python.exe -m britecore_mcp.server
```

### Best first exploration target

```sql
SELECT *
FROM v_logical_catalog;
```

Use this to locate the logical views and fields that match the business question.

## `britecore_sdk`

### Local editable install

```powershell
cd C:\PythonProjects\BriteCore\britecore_sdk
python -m pip install -e .
```

### Optional extras

```powershell
python -m pip install -e ".[async-http]"
python -m pip install -e ".[typed-config]"
```

### Typical use pattern

```python
from britecore_sdk.api.api_calls import get_api_client
from britecore_sdk.api.api_calls.v2 import policies

client = get_api_client()
result = policies.retrieve_policy(policy_number="POL001")
print(result)
```

## `britecore-csv-loader`

### Local install

```powershell
cd C:\PythonProjects\data\britecore-csv-loader
python -m pip install -e .
```

### Relationship inspection

```powershell
python -m britecore_csv_loader.cli --relationships
```

### Claims extractor example

```powershell
python -m britecore_csv_loader.part2_claims --reporting-year 2025
```

### Python usage

```python
from britecore_csv_loader import RawCsvInterface

api = RawCsvInterface()
revision_id = api.load_dataset("revisions")[0]["revisionId"]
print(api.revision_summary(revision_id))
print(api.extract_entity_links(revision_id))
```

## `PolicyReporting`

### Typical setup pattern

```powershell
cd C:\PythonProjects\PolicyReporting
python -m pip install -e .
```

### Query pattern

```python
from policy_reporting import CrossSourcePolicyQuery

query = CrossSourcePolicyQuery.from_env()
result = query.find_by_form("HO 002*")
print(result.summary())
```

## Cross-project starter workflow

For most reporting work, the best flow is:

1. Identify the business object and grain.
2. Use `britecore_mcp` to find logical layers and candidate views.
3. Verify relationship assumptions from raw CSV exports if needed.
4. Use `britecore_sdk` for live API retrieval when required.
5. Move the final cross-source logic into `PolicyReporting` when a service boundary is needed.

## Practical decision guide

### If the ask is "Which table has this data?"

Use `britecore_mcp`.

### If the ask is "Can you pull this from the live API?"

Use `britecore_sdk`.

### If the ask is "I only have raw exported CSVs and need the relationships?"

Use `britecore-csv-loader`.

### If the ask is "I need one service interface across BriteCore and other sources?"

Use `PolicyReporting`.

## Documentation references

- [Architecture](architecture.md)
- [Source Registry](source_registry.md)
- [Project Map](project_map.md)
- [Reporting Logical Layer](reporting/logical_layer.md)
- [Reporting Playbook](reporting/reporting_playbook.md)
