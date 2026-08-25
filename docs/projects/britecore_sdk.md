# britecore_sdk

## Role

Python SDK for the BriteCore Insurance API.

## What the project does

Based on the local `README.md`, `GETTING_STARTED.md`, and `ARCHITECTURE.md`, `britecore_sdk` is a production-oriented Python SDK that provides:

- spec-aligned API wrappers
- API key and OAuth2 authentication flows
- lazy client initialization
- sync and async client patterns
- typed models, validators, and reusable configuration management

## Best-known documentation entry points

- `C:\PythonProjects\BriteCore\britecore_sdk\README.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\GETTING_STARTED.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\ARCHITECTURE.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\API.md`
- `C:\PythonProjects\BriteCore\britecore_sdk\CONFIG_MANAGEMENT.md`

## What it is strongest at

- authenticated API access
- reusable Python client integrations
- endpoint wrappers and typed models
- configuration management and auth flows

## Key local documentation themes

- quick start and editable install workflows
- obtaining credentials from the BriteCore UI
- config-file and environment-variable precedence
- domain/API/infrastructure layering in the SDK architecture
- troubleshooting and optional extras for async or interactive usage

## Recommended use cases

Use `britecore_sdk` first when:

- you need live API access instead of report-layer SQL or raw exports
- you want reusable Python code around BriteCore endpoints
- you need a supported auth/config story for BriteCore integrations
- you want to automate validation or enrichment alongside reporting tools

## Relationship to other projects

- complements `britecore_mcp` when API automation needs to accompany report analysis
- can feed data or orchestrations into `PolicyReporting`
- is separate from `britecore-csv-loader`, which works with exported CSV datasets rather than live API calls

