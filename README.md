# britecore_docs

<p align="center">
  <img src="https://raw.githubusercontent.com/sshimek42/britecore_docs/main/.assets/britecore-docs-banner.svg" alt="BriteCore Docs banner" width="100%" />
</p>

[![Public Repo](https://img.shields.io/badge/public-repo-3b82f6?logo=github)](https://github.com/sshimek42/britecore_docs)
[![Docs Site](https://img.shields.io/badge/docs-GitHub%20Pages-10b981?logo=github)](https://sshimek42.github.io/britecore_docs/)
[![MkDocs](https://img.shields.io/badge/build-MkDocs-0f766e?logo=markdown)](https://www.mkdocs.org/)

A shared documentation and architecture hub for BriteCore reporting, lineage, and data-model patterns.

This repository is intentionally a synthesis layer: it explains how the wider BriteCore ecosystem fits together, where the source-of-truth lives, and how to work across raw exports, API access, and SQL/reporting layers without duplicating implementation detail in the owning projects.

## What this project covers

- ecosystem architecture and repo boundaries
- source-of-truth ownership and routing guidance
- CSV, API, and SQL/report access patterns
- data lineage, grain, joins, and bridge relationships
- domain references for policies, claims, contacts, properties, payments, and related objects
- onboarding and workflow guidance for cross-project work

## What this project is not

This repo is not the canonical home for:

- report implementation code
- runtime application logic
- raw loader or warehouse code
- generated operational artifacts

Those details remain in the source repos that own them.

## Working model

The goal of this project is to make the broader system easier to understand:

- identify the correct source repo for a task
- explain how the same business object appears in different access modes
- document the relationship patterns that are easy to miss across CSV, API, and SQL/report layers
- preserve a clear boundary between implementation code and shared ecosystem knowledge

## Structure

```text
britecore_docs/
  README.md
  mkdocs.yml
  docs/
    index.md
    architecture.md
    onboarding.md
    quickstarts.md
    source_of_truth_matrix.md
    csv_api_sql_mapping.md
    access_mode_decision_matrix.md
    bridge_table_guide.md
    domain_reference.md
    ai_workflow_guide.md
```

## Recommended usage

- Start here for ecosystem understanding, repo selection, and cross-project workflows.
- Follow the owning project docs for implementation details and canonical examples.
- Use this repo as the bridge and reference layer, not as a second copy of production logic.

