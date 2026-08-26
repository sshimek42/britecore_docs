# Source-of-Truth Matrix

This page is the short answer to a recurring question in the BriteCore ecosystem: which repo owns the canonical details for a given task?

The rule is simple:

- If the question is about source-system behavior, start in the source project.
- If the question is about how the ecosystem fits together, use this repo.
- If the question is about a specific operational implementation, go to the repo that owns that implementation.

## Canonical ownership by concern

| Concern | Canonical owner | Why it is authoritative |
|---|---|---|
| MCP runtime behavior and logical-layer discovery | `britecore_mcp` | Owns the interactive toolchain and logical view patterns |
| SQL report definitions and validation SQL | `britecore_sql_reports` | Owns the report specs and report metadata |
| SDK/API access patterns and resource relationships | `britecore_sdk` | Owns the client, auth, config, endpoint contracts, and resource-level relationship definitions |
| Cross-source reporting service logic | `PolicyReporting` | Owns the service-layer assembly and report-object logic |
| Raw CSV structure and relationship reconstruction | `britecore-csv-loader` | Owns CSV ingestion, `*Id` linkage, and revision snapshots |
| Warehouse landing, normalization, and report-ready views | `policy-data-warehouse` | Owns raw/core/reporting schema boundaries and loader logic |
| Regulatory extraction and BriteCore-to-NAIC translation | `NAIC` | Owns the report-generation and mapping logic |
| This docs hub | `britecore_docs` | Explains the ecosystem, not the implementation |

## API docs are often the best resource-relationship guide

For entity relationships, the SDK API docs and endpoint modules are often the clearest source of truth for:

- which resource groups exist (`contacts`, `claims`, `policies`, `payments`, etc.)
- which request parameters are required for a lookup
- which ids are returned by a call and therefore become the next lookup key
- which resource domain is the canonical place for a relationship lookup

This is especially useful before translating from nested API objects into flattened SQL/report views. The API wrapper signatures themselves are a practical map of the chain.

## The same pattern is consistent across the whole SDK

This is not limited to claims, contacts, and policies. The v2 SDK modules under `src/britecore_sdk/api/api_calls/v2/` follow the same broad contract across the entire product surface.

Examples:

- `files.py`: `generate_download_url(file_id=...)`, `retrieve_active_file_objects(date_added_from=..., label=...)`
- `drivers.py`: `create_driver(driver)`, `get_driver(id)`, `list_drivers_for_quote(quote_id)`, `update_driver(driver)`, `delete_driver(id)`
- `vehicles.py`: `create_vehicle(vehicle)`, `get_vehicle(id)`, `list_vehicles_for_quote(quote_id)`, `delete_vehicle(id)`
- `claims.py`: `get_claim(claim_id=...)`, `get_claim_contacts(claim_id=...)`, `get_claim_payments(claim_id=...)`
- `policies.py`: `retrieve_policy(policy_number=...)`, `retrieve_policy_ids(policy_number=...)`, `retrieve_policy_terms(policy_id=...)`, `retrieve_revision_details(revision_id=...)`

The pattern is usually:

1. list the things belonging to a parent object or quote
2. get one object by id or by a stable business key
3. fan out to relationship-specific calls using the returned id
4. create/update/delete through the same domain contract

This means the ecosystem can often be linked end-to-end by following the request/response ids and parent-child relationships, not just by a handful of headline entities.

## Practical interpretation

### Use this repo when you need:

- a mental model of the whole stack
- repo-role guidance
- task-based routing
- cross-project terminology
- bridge knowledge not owned in a single implementation repo

### Do not use this repo as the implementation source-of-truth for:

- raw SQL
- loader logic
- report SQL definitions
- API patterns
- CSV extraction logic
- runtime MCP behavior

## Helpful shorthand

If the question is:

- “What is the raw data shape?” → `britecore-csv-loader`
- “What does the warehouse do?” → `policy-data-warehouse`
- “How do I get a report-ready answer across sources?” → `PolicyReporting`
- “How do I produce a NAIC report?” → `NAIC`
- “What is the base entity naming pattern?” → `britecore-csv-loader` + `csv_api_sql_mapping.md`
- “How does this ecosystem fit together?” → `britecore_docs`

## Base-entity naming rule

For base entities, the raw CSV name is usually the best first clue to the entity-level SQL/report view name.

Examples:

- `Contacts.csv` → `v_contacts`
- `Claims.csv` → `v_claims`
- `Properties.csv` → `v_properties`
- `Revisions.csv` → `v_revisions`

This is a strong rule for base entities and entity master tables, but not for many-to-many bridge views such as `v_claims_contacts` and `v_revisions_contacts`, which are relationship views rather than direct raw-file mirrors.

## Consolidation rule for this docs repo

When a fact is useful to more than one project but not owned by any one implementation repo, it belongs here as a synthesized bridge document.

Good examples:

- raw-to-warehouse-to-report flow
- BriteCore status/cancellation semantics
- revision/policy/claim lineage
- repo-selection guidance by task

This keeps the project useful without becoming a second implementation repo.
