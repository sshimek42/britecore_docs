# CSV / API / SQL Mapping Overview

This page captures the observed mapping pattern across the BriteCore stack:

- raw CSV names are usually close to the underlying entity names
- the API returns a nested JSON object for that entity
- the SQL/report layer is often a denormalized, flattened presentation of the same business object

## High-level mapping pattern

```text
Raw CSV file              -> base entity / raw-source view
Contacts.csv              -> contact universe / contact entity
Revisions.csv             -> revision history / revision entity
Claims.csv                -> claim master records
Properties.csv            -> property records
...

API response object       -> nested entity payload
SQL report / v_* view     -> flattened reporting projection
```

## Important operational reality

These are not a strict dependency chain that must always be used together.

A team may legitimately work with:

- only the raw CSV exports
- only the API payloads
- only the SQL/report views
- or some combination of the three

The real question is not “which one is the true source?” but rather:

- what do I have access to?
- how much shape and aggregation do I want the system to do for me?
- am I looking for raw facts, nested domain objects, or a report-ready projection?

In practical terms:

- CSV is best when you want point-in-time stored export data and offline access to a specific extraction set
- API is best when you want real-time or near-real-time access to a structured object model, and the SDK wrappers show the exact parameters and ids needed to walk the object graph
- SQL/reporting is best when you want a preformatted, query-friendly layer that is easy to build reports from

In many basic cases, a direct SQL query from the website is also the easiest path because the reporting layer already contains the flattened joins and common business fields. This can be much simpler than reconstructing the same answer from several CSV exports or a long sequence of API calls.

In other words, the three layers are parallel access modes, not a mandatory pipeline. The SDK API docs are especially valuable for mapping chains because the wrapper signatures make the request and ID flow explicit.

This same pattern is not limited to claims, contacts, and policies. The v2 modules under `src/britecore_sdk/api/api_calls/v2/` all follow a consistent resource contract: list/search by parent or scope, get/retrieve by id or stable key, create/update/delete by payload, and relationship-specific calls for the attached collections. Examples include `files.py`, `drivers.py`, `vehicles.py`, `claims.py`, and `policies.py`.

## The practical rule

For many BriteCore entities, the base CSV filename maps closely to the logical SQL table/view name.

Examples:

- `Contacts.csv` ≈ `v_contacts`
- `Revisions.csv` ≈ `v_revisions`
- `Claims.csv` ≈ `v_claims`

This is not always a literal one-to-one match for bridge or relationship views, but it is a strong rule for the base entity tables.

## Field naming pattern: API and SQL are usually closer to each other than CSV is

A very strong practical pattern is that the SQL/report field names and the API object keys usually align more closely than either does with the raw CSV column names.

Examples:

- CSV: `contactId`
- API: `id`
- SQL: `contact_id`

- CSV: `policyNumber`
- API: `policy_number`
- SQL: `policy_number`

- CSV: `claimNumber`
- API: `claim_number`
- SQL: `claim_number`

- CSV: `revisionId`
- API: `revision_id`
- SQL: `revision_id`

The CSV layer usually preserves the export naming pattern and field formatting from the source system, while the API and report layer tend to normalize to a more stable snake_case / logical-view naming convention.

## CSV files are often intentionally split by concern

The CSV export pattern is similar to the API pattern, but more fragmented. The raw export tends to separate the master record from the related detail tables rather than nesting everything into one payload.

Examples from `C:\PythonProjects\data\britecore-csv-loader\data\mow\raw` show this clearly:

- `claims.csv` contains the master claim record
- `claims_contacts.csv` contains many-to-many claim-to-contact linkage
- `claims_payments.csv` contains claim payment records
- `claims_payments_item_amounts.csv` contains the line-item detail for each payment
- `policies.csv` contains the policy master record
- `policy_terms.csv` contains the term/revision-specific information
- `addresses.csv`, `phones.csv`, `emails.csv`, and `mortgagees.csv` split out child objects that the API may return as nested arrays
- `item_questions.csv` and `builder_obj_parsed.csv` show sub-entity and form-detail data separated out further

That is the key difference from the API layer: CSVs are often more normalized and more spread across multiple files, so reconstructing a single business record can take more joins or chained lookups than a single API call.

## Contacts example

The raw CSV `contacts.csv` contains the direct entity fields and the row looks like a contact master record:

```text
contactId: 00025e4c-af2e-4a1e-bdc6-3808ae83e7ce
name: Kelly Sobojinski
type: individual
username: ...
website: ...
terminationReason: ...
```

The API payload for the same contact is a nested object rather than a flat row:

```json
{
  "id": "00025e4c-af2e-4a1e-bdc6-3808ae83e7ce",
  "name": "Kelly Sobojinski",
  "type": "individual",
  "addresses": [{ "city": "Oshkosh", "state": "WI", "zipcode": "54904" }],
  "phones": [{ "phone": "1-920-216-2355", "type": "Home" }],
  "roles": [{ "name": "Named Insured" }]
}
```

The SQL report output is then flattened into a denormalized reporting row:

```text
contact_id = 00025e4c-af2e-4a1e-bdc6-3808ae83e7ce
contact_name = Kelly Sobojinski
contact_type = individual
address_city = Oshkosh
address_state = WI
address_zip = 54904
primary_phone = 1-920-216-2355
roles = Named Insured
```

This pattern matters because it explains why the same record appears in three forms:

1. raw CSV: a base entity record with direct fields
2. API: nested resource object with arrays and child objects
3. SQL/report: flattened, business-facing projection of the entity plus common related fields

## Why this matters

If you assume the API object is the same shape as the CSV row, you will miss the denormalization and nesting that happens in the API contract. If you assume the report view is the same as the raw CSV, you will miss the flattening and enrichment that happens in SQL/reporting.

## Recommended documentation rule

A good working rule is:

- raw CSV ≈ base entity shape
- API ≈ nested domain object
- SQL report / `v_*` view ≈ flattened business projection

For entity-level work such as `contacts`, `claims`, `policies`, `properties`, and similar, the CSV name is often the best first clue to the logical table/view name.

## Important exception

Relationship or bridge tables may not correspond 1:1 to a CSV file. For example:

- `v_claims_contacts` is a bridge relationship view
- `v_revisions_contacts` is a bridge relationship view

Those are not necessarily raw CSV files. They are created to make many-to-many or join semantics tractable in reporting.

## Short version

Base entity CSVs are usually close to the `v_*` report tables. The API adds nesting, and the report layer flattens and enriches the entity for consumption.
