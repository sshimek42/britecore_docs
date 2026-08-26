# AI Workflow Guide for Other Repos

This page is the operating guide for using AI effectively when working in the other BriteCore projects.

## Primary rule: keep implementation in the owning repo

The owning repo is the source of truth for the actual code, APIs, SQL, and generation logic.

Examples:

- `britecore_sdk` owns SDK implementation and endpoint behavior
- `britecore_sql_reports` owns report definitions and report docs
- `britecore_mcp` owns MCP runtime behavior and logical discovery workflows
- `britecore-csv-loader` owns raw CSV extraction logic
- `policy-data-warehouse` owns warehouse landing and normalization logic
- `NAIC` owns regulatory mapping/report generation

This docs repo should be used for:

- cross-project mapping
- source-of-truth routing
- bridge knowledge
- ecosystem architecture
- issue framing and task planning

It should not be used as the implementation home for logic that belongs in the owning project.

## When AI is working in a repo

Use this rule of thumb:

1. start in the repo that owns the behavior
2. inspect the local implementation docs first
3. keep the patch focused on the owning codebase
4. use `britecore_docs` to clarify architecture, lineage, and cross-project mapping
5. do not duplicate implementation details in this repo unless they are truly reusable ecosystem guidance

## Good AI prompts for this repo family

A strong prompt should be explicit about repo boundaries.

Examples:

- “I’m working in `britecore_sdk`. Use the SDK as the source of truth for API behavior, and only use `britecore_docs` to explain cross-project mapping.”
- “This is a report engineering task in `britecore_sql_reports`; do not duplicate the SQL into `britecore_docs` unless we need a synthesized bridge explanation.”
- “This is a CSV reconstruction task in `britecore-csv-loader`; document the raw extraction logic there and keep `britecore_docs` focused on the ecosystem mapping.”

## The decision model for AI work

Use this decision path:

- if the task is implementation: do it in the owning repo
- if the task is architecture or ecosystem understanding: do it in `britecore_docs`
- if the task is troubleshooting across repos: use the owning repo for implementation, then use this repo for integration context

## What to avoid

Avoid these patterns:

- copying raw SDK code or SQL into `britecore_docs` as if it were the canonical home
- creating parallel implementations of logic in the docs repo
- documenting every repo as a full duplicate of the same concept
- treating the docs repo as a second codebase

## Recommended documentation behavior for AI-assisted tasks

When adding documentation in the repos, prefer:

- repo-owned implementation notes
- short operational docs in the owning repo
- high-level cross-project explanations in `britecore_docs`
- bridge docs that explain how repos connect, not full copies of implementation details

## Short version

Use AI to stay disciplined:

- implementation in the real repo
- explanation and mapping in the docs repo
- no unnecessary duplication
- no “second home” for logic that belongs elsewhere

This keeps the ecosystem maintainable and prevents the docs repo from drifting into implementation duplication.
