# britecore_sdk

## Role

`britecore_sdk` is the live API access layer for BriteCore. It is the project to use when you want to talk directly to the operational BriteCore system in Python rather than working from a logical catalog, report SQL, or exported CSV snapshot.

## What the SDK does well

Based on the local SDK docs, the project provides:

- spec-aligned API wrappers
- API key and OAuth2 authentication flows
- lazy client initialization
- sync and async client patterns
- typed models, validators, and reusable configuration management
- a production-oriented Python interface for live API automation

## Where it sits in the stack

```text
API layer:                    britecore_sdk
Logical discovery:            britecore_mcp
SQL/report definition layer:  britecore_sql_reports
CSV export reconstruction:    britecore-csv-loader
Service/report staging:       PolicyReporting
Docs/architecture map:        britecore_docs
```

`britecore_sdk` is the canonical entry point when a workflow requires a live call to BriteCore. It is the operational API layer, not the logical/reporting abstraction.

## Recommended use cases

Use `britecore_sdk` when you need:

- live policy, claim, quote, or contact data from the BriteCore API
- Python automation against authenticated endpoints
- a supported auth/config story for runtime integrations
- validation or enrichment work that must reflect current system state
- custom orchestration that sits above report outputs or CSV extracts

Use `britecore_mcp` when you want:

- logical discovery of tables, fields, and metadata
- report-oriented introspection and catalog context
- runtime guidance for how a logical layer is surfaced

Use `britecore_sql_reports` when you need:

- report definitions
- join starters and grain guidance
- SQL-based analysis and validation

## Authentication and configuration model

The SDK documentation emphasizes configuration-first setup. Common patterns include:

- API key auth
- OAuth client auth
- environment-driven configuration for CI and runtime automation
- config-file precedence across user/project/local settings

The main design idea is that credentials and site selection are driven by configuration, not hardcoded in application code.

Typical configuration inputs:

- `target_site`
- `base_url`
- credentials for API-key or OAuth mode
- environment overrides when running in CI or production

## Minimal usage pattern

The SDK now supports lazy shared client initialization, plus explicit per-client initialization when you want full control over connection state. The canonical pattern is either `init_api_client(...)` followed by `get_api_client()` or a direct `BritecoreAPIClient(...).init_client()`.

```python
from britecore_sdk.api.api_calls import get_api_client, init_api_client
from britecore_sdk.api.api_calls.v2 import policies

# Shared module client (configured target site or env/settings-backed auth)
init_api_client(target_site="production")
client = get_api_client()
result = policies.retrieve_policy(policy_number="POL001")
print(result)
```

Explicit client pattern:

```python
from britecore_sdk.api.britecore_api_client import BritecoreAPIClient
from britecore_sdk.api.api_calls.v2 import policies

client = BritecoreAPIClient("production").init_client()
result = policies.retrieve_policy(policy_number="POL001", client=client)
print(result)
```

## Live API vs logical/reporting workflows

The SDK should be thought of as the tool for direct system access. It complements, rather than replaces, the other projects:

- `britecore_sdk` gives live API responses
- `britecore_mcp` gives logical and metadata context
- `britecore_sql_reports` gives report-authoring and SQL validation context
- `britecore-csv-loader` reconstructs exports and lineage from raw CSV data

This distinction matters because a live API response is not the same as a logical view, and a report SQL result is not the same as a direct endpoint payload.

## Canonical documentation home

`britecore_sdk` owns its implementation and usage documentation in the SDK repo itself. `britecore_docs` is a summary layer for ecosystem context; it should not duplicate the SDK’s operational docs.

Best-known entry points in the SDK repo:

- `C:\PythonProjects\BriteCore\britecore_sdk\README.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\GETTING_STARTED.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\ARCHITECTURE.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\API.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\CONFIG_MANAGEMENT.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\TROUBLESHOOTING.md`

This page should stay at the "how this fits into the ecosystem" level. Detailed installation, auth, config, auth flows, examples, and troubleshooting belong in the SDK project itself.

## Fast local setup pattern

```powershell
cd C:\PythonProjects\BriteCore\britecore_sdk
python -m pip install -e .
```

Optional extras are available when needed:

```powershell
python -m pip install -e ".[async-http]"
python -m pip install -e ".[typed-config]"
```

## Relationship to other projects

- complements `britecore_mcp` when API automation needs to accompany report analysis
- can feed data or orchestrations into `PolicyReporting`
- is separate from `britecore-csv-loader`, which works with exported CSV datasets rather than live API calls
- is explicitly not the owner of SQL reporting definitions or report cache metadata

## Recommendation for the SDK project itself

The SDK project should keep emphasizing these points in its own docs:

1. It is the API layer, not the data warehouse/reporting layer.
2. It is configured around site-specific auth and environment settings.
3. It is designed for real-time automation and direct system access.
4. It works best when paired with MCP or report tools for discovery and analysis.
5. Its role in the BriteCore ecosystem is to provide a consistent Python interface to live platform data, not to be a replacement for logical metadata or SQL reporting artifacts.

