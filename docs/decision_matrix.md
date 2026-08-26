# Decision Matrix

Use this page when choosing where to start on a BriteCore reporting or integration task.

## Quick triage

| Question type | Best starting repo | Why |
|---|---|---|
| Which table or field contains the data? | `britecore_mcp` | Best reporting discovery and logical catalog guidance |
| Can I pull this from the live API? | `britecore_sdk` | API access and client patterns live here |
| I only have raw CSV exports and need to understand relationships | `britecore-csv-loader` | Best raw export lineage and relationship analysis |
| I need one cross-source reporting boundary | `PolicyReporting` | Best service-layer and multi-source design |
| I need to understand the overall ecosystem | `britecore_docs` | Shared architecture and cross-project guidance |

## Decision tree

### 1. Reporting semantics

If the question is about:

- table names
- view discovery
- field descriptions
- join logic
- row grain
- logical reporting model

Then start with `britecore_mcp` and consult `v_logical_catalog`.

### 2. Live API access

If the question is about:

- authenticated API calls
- Python client usage
- runtime configuration
- endpoint wrappers and automation

Then start with `britecore_sdk`.

### 3. Raw exports and lineage

If the question is about:

- CSV relationships
- exported entity matching
- claim or policy snapshots
- revision-based links across raw files

Then start with `britecore-csv-loader`.

### 4. Shared reporting service

If the question is about:

- combining multiple sources into a single output
- service and query orchestration
- reporting APIs or downstream consumers
- higher-level report objects

Then start with `PolicyReporting`.

## Common report assembly flow

```text
Business question
    -> Identify anchor object
    -> Discover logical views
    -> Check grain and join assumptions
    -> Validate against raw exports if needed
    -> Use API if live access is required
    -> Move to service layer if cross-source orchestration is needed
```

## Working patterns by task type

### New report design

- Start with `britecore_mcp`
- Use `v_logical_catalog`
- Validate grain and join patterns
- Add raw export validation if needed

### Existing report debugging

- Recheck the anchor object and grain
- Inspect the join fan-out
- Compare the view logic with export lineage
- Confirm whether the issue is in source semantics or output assembly

### Integration work

- Use `britecore_sdk` for API calls
- Use `PolicyReporting` for shared service/workflow assembly
- Keep repo-specific implementation details in the owning repo

### Onboarding work

- Read `architecture.md`
- Read `project_map.md`
- Read `onboarding.md`
- Use `quickstarts.md` to get started quickly

## Rules of thumb

- If the task is about the business meaning of the data, start in `britecore_mcp`.
- If the task is about live system data, start in `britecore_sdk`.
- If the task is about exported file structure, start in `britecore-csv-loader`.
- If the task is about a shared reporting product, start in `PolicyReporting`.
- If the task is about team or process alignment, start in `britecore_docs`.
