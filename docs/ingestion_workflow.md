# Ingestion Workflow

This page describes how to pull documentation into `britecore_docs` without losing source-of-truth discipline.

## Preferred approach

### Phase 1: Curated linking

Start by creating overview pages that:
- summarize a source repo
- point to its key canonical documents
- explain how it fits with the other BriteCore projects

This avoids duplicating fast-changing implementation details.

### Phase 2: Selective mirroring

Mirror content into `britecore_docs` only when:
- the content is cross-project in nature
- it needs common formatting/navigation
- it is stable enough to maintain centrally
- it adds value beyond a simple link

### Phase 3: Consolidated playbooks

Create net-new docs in `britecore_docs` for:
- cross-repo architecture
- reporting decision trees
- end-to-end workflows
- glossary and conventions
- onboarding guides

## Suggested import candidates

### From `britecore_mcp`
- report table framework
- report join starters
- logical-layer reporting overview
- selected high-value report table references

### From `britecore_sdk`
- SDK quick start summary
- auth/configuration overview
- architecture summary
- API usage patterns

### From `PolicyReporting`
- service-layer purpose and architecture
- environment and SQL backend setup
- cross-source query patterns

### From `britecore-csv-loader`
- raw CSV relationship overview
- revision-centric snapshot model
- CLI and extractor workflows
- relationship-diagram references

## Maintenance guidance

For mirrored content, add a source note such as:

```markdown
Source of truth: `C:\path\to\original\file.md`
Last reviewed: YYYY-MM-DD
```

## Workspace recommendation

For active consolidation work, add these folders to the same IDE workspace:

- `C:\PythonProjects\BriteCore\britecore_docs`
- `C:\PythonProjects\BriteCore\britecore_mcp`
- `C:\PythonProjects\BriteCore\britecore_sdk`
- `C:\PythonProjects\PolicyReporting`
- `C:\PythonProjects\data\britecore-csv-loader`

