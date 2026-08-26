# Common Gotchas and Anti-Patterns

This page is the practical list of mistakes to avoid when working with BriteCore data across CSV, API, and SQL/report layers.

The goal is not to list every possible issue; it is to capture the recurring traps that cause confusion when mapping records across the system.

## 1. Treating CSV, API, and SQL as if they were the same exact shape

This is the most common mistake.

A raw CSV row, API object, and report-view row can all represent the same business record, but they are not the same structure.

Examples:

- `contacts.csv` is a flat record export with raw field names
- `contacts.get_contact(...)` returns nested arrays like `addresses`, `phones`, `emails`, and `roles`
- `v_contacts` is a flattened reporting projection and may include fields such as `primary_phone`, `full_address`, and `roles`

The same object can appear as:

- a flat row
- a nested document
- a denormalized report view

These are parallel access patterns, not interchangeable schemas.

## 2. Assuming one-to-one joins everywhere

A lot of BriteCore relationships are one-to-many or many-to-many.

Examples:

- one claim can have many contacts
- one revision can have many agencies or contacts
- one payment can have many item amounts
- one contact can be associated with multiple claims or revisions

This means a base entity lookup can easily expand into several rows when you join to bridge or child tables. That is not necessarily a duplicate; it is often normal relationship multiplicity.

## 3. Mistaking bridge tables for base tables

Relationship tables such as:

- `v_claims_contacts`
- `v_revisions_contacts`
- `v_revisions_agencies`
- `v_claim_payments_item_amounts`

should be treated as relationship or bridge tables, not as master entity tables.

If you see a result set multiplying because the same parent id is repeated in many rows, the likely cause is a bridge table, not duplicate master data.

## 4. Using a human-readable key as if it were the internal key everywhere

The business key and the internal key are not always the same thing.

Examples:

- `policy_number` leads to `revision_id`
- `claim_number` leads to `claim_id`
- `contact_id` is often the internal anchor for addresses, phones, and role records

The key principle is:

- human-facing key for lookup
- internal id for relationship traversal

If you try to join directly on `policy_number` in every table, you will often fail because the detailed tables are keyed by `revision_id`, `claim_id`, or `contact_id` instead.

## 5. Ignoring the fact that CSVs are often split by concern

The CSV export is usually more fragmented than the API object.

You may need to chain several files to reconstruct a single business record:

- `claims.csv` + `claims_contacts.csv` + `claims_payments.csv`
- `policies.csv` + `policy_terms.csv` + `revisions.csv` + `properties.csv`
- `contacts.csv` + `addresses.csv` + `phones.csv` + `emails.csv`

If you assume the CSV layer behaves like a single nested object model, the join logic will be wrong and the missing links will look like data gaps.

## 6. Assuming SQL/report views are always “the answer”

The report layer is convenient and query-ready, but it is not always the best source for all tasks.

Use SQL/report views when:

- you need a fast answer
- you want a flattened reporting view
- you need to build or extend a report quickly

Use CSV when:

- you need raw-source fidelity
- you need point-in-time validation
- you want to inspect extraction behavior

Use API when:

- you want the current entity object model
- you want nested child arrays
- you need to understand the live relationship structure

## 7. Forgetting that field naming changes across layers

The same concept may appear with different casing or formatting across sources.

Examples:

- CSV: `contactId`
- API: `id` or `contact_id`
- SQL: `contact_id`

- CSV: `claimNumber`
- API: `claim_number`
- SQL: `claim_number`

- CSV: `revisionId`
- API: `revision_id`
- SQL: `revision_id`

The practical rule is: do not trust the raw export naming alone. Match on the logical identity and use the ID contract to cross-check.

## 8. Confusing current-state data with historical-change data

Some tables represent current state; others represent revision history or change logs.

Examples:

- a policy revision can show the state at a particular term
- a change log can show modifications over time
- a contact may have multiple historical relationship records across policy terms

If you do not distinguish current-state vs historical-state logic, your counts and relationships will be misleading.

## 9. Treating all date fields as the same kind of date

Not all date fields are directly comparable.

Examples:

- effective date
- expiration date
- cancellation date
- commit date
- date reported
- date updated
- date added

Some are business lifecycle dates, some are system timestamps, some are raw export strings. The exact meaning must be checked before using them for trend analysis or KPI logic.

## 10. Ignoring the role and tag text fields

Role and tag fields such as:

- `roles`
- `systemTags`
- `claimSystemTags`

are often denormalized text values rather than normalized relationship tables.

That means:

- role labels may be comma-separated
- tags may be stored as text payloads
- you may need to parse or normalize before using them in business logic

## 11. Overfitting to a single entity example

The pattern is general across the ecosystem, not only for claims and contacts.

The same logic applies to:

- policies and revisions
- properties and mortgagees
- claims and payment detail
- files and attachments
- drivers, vehicles, and other nested domains

If a solution works for policies but not for claims, it usually means the query is anchored to the wrong key or wrong grain.

## 12. Forgetting that bridge joins are often the real answer

When you are stuck, the usual fix is not “there is no data.” It is usually:

- find the base entity id
- join to the bridge table
- then join to the related master table

This is the most common pattern in the BriteCore ecosystem and the most reliable workflow for reconstruction.

## Best practice checklist

Before concluding that a record is missing, check:

- Did I start from the right business key?
- Did I resolve it to the correct internal id?
- Am I on the base entity or a bridge table?
- Is the CSV split by concern rather than nested?
- Is the API object nested and the SQL layer flattened?
- Am I using the correct grain for the relationship?

## Short version

The recurring anti-patterns are:

- assuming same shape across CSV/API/SQL
- assuming one-to-one joins
- treating bridge tables as master tables
- using raw export names without mapping to ids
- ignoring fragmentation in CSV exports
- forgetting that relationship multiplicity is a normal part of the system

If you keep those in mind, most BriteCore data reconciliation issues become much easier to diagnose.
