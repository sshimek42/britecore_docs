# britecore_docs

Cross-project documentation hub for BriteCore-related tooling and reporting assets.

## Purpose

`britecore_docs` is a documentation-only project intended to consolidate and organize guidance that currently lives across multiple repositories and tools, including:

- `C:\PythonProjects\BriteCore\britecore_mcp`
- `C:\PythonProjects\BriteCore\britecore_sdk`
- `C:\PythonProjects\PolicyReporting`
- `C:\PythonProjects\data\britecore-csv-loader`

The goal is not to replace canonical source docs immediately, but to provide:

- a single starting point
- cross-project architecture and workflow guides
- reporting and join guidance
- source maps back to the authoritative repo/location
- a stable place to curate higher-level documentation that spans repos

## Initial scope

This scaffold starts with:

- a source registry
- a project map
- per-project overview pages
- a reporting/logical-layer overview
- an ingestion workflow for bringing docs in from other repos

## Proposed structure

```text
britecore_docs/
  README.md
  mkdocs.yml
  docs/
    index.md
    source_registry.md
    project_map.md
    ingestion_workflow.md
    reporting/
      logical_layer.md
      join_starters.md
    projects/
      britecore_mcp.md
      britecore_sdk.md
      policy_reporting.md
      britecore_csv_loader.md
```

## Source-of-truth philosophy

Until content is formally migrated, treat the original repositories as canonical:

- `britecore_mcp` remains canonical for MCP server behavior and report-table docs
- `britecore_sdk` remains canonical for SDK API/configuration/architecture docs
- `PolicyReporting` remains canonical for cross-source reporting service docs
- `britecore-csv-loader` remains canonical for raw CSV relationships, ingestion, and extractor workflows

`britecore_docs` should initially act as a curated layer that links to, summarizes, and eventually standardizes those sources.

## Suggested next steps

1. Add `C:\PythonProjects\BriteCore\britecore_docs` to your JetBrains workspace.
2. Optionally add these sibling projects to the same workspace:
   - `C:\PythonProjects\BriteCore\britecore_sdk`
   - `C:\PythonProjects\PolicyReporting`
   - `C:\PythonProjects\data\britecore-csv-loader`
3. Decide whether this repo should:
   - mirror selected docs by copy/sync, or
   - only index/link back to source repos
4. If desired, initialize a standalone git repository here and publish it separately.

