# Record Trace Workflow

This page is the practical “how do I actually trace one record?” guide.

## Core principle

The exact connection path varies by entity and by system, but the same pattern shows up repeatedly:

- a human-facing identifier such as `policy_number` or `claim_number` leads to a stable internal id
- that internal id is then used to find the revision or entity relationships across the raw CSV, API, and reporting layers
- the relation is often strongest through `revision_id`, then through bridge views or nested API objects

This is not a strict universal rule for every record, but it is the dominant pattern in the BriteCore ecosystem.

## Policy trace pattern

A policy usually starts from:

- `policy_number` in the raw CSV or `v_revisions.policy_number`
- then resolves to a `revision_id`
- then the `revision_id` becomes the main anchor for related policy, claim, contact, and property lineage

This is consistent with the reporting docs:

- `v_revisions` is one row per revision (`revision_id`)
- `v_revisions.revision_id` underpins joins to `v_claims`, `v_drivers`, `v_revisions_contacts`, and `v_revisions_agencies`

### Policy trace path

```text
policy_number
    -> revision_id (via revision or policy-term record)
    -> related claims, contacts, properties, etc.
```

### Example SQL pattern

```sql
SELECT r.revision_id, r.policy_number, c.claim_id, c.claim_number
FROM v_revisions r
LEFT JOIN v_claims c
  ON c.revision_id = r.revision_id
WHERE r.policy_number = 'YOUR_POLICY_NUMBER';
```

This is the general idea: follow the policy back to the revision, then fan out from the revision to other related objects.

## Claim trace pattern

Claims follow a similar structure:

- `claim_number` is the human-facing identifier
- `claim_id` is the internal row id
- `revision_id` links the claim back to the policy term or revision context
- `claim_id` then links to claim detail and bridge tables such as contacts, payments, items, disputes, and dates

This is also documented in `v_claims`:

- key columns include `claim_id`, `claim_number`, `policy_number`, and `revision_id`
- `claim_id` is the central join key for `v_claim_items`, `v_claim_payments`, `v_claim_change_log`, `v_claims_disputes`, `v_claims_perils`, `v_claims_dates`, and `v_claims_contacts`

### Claim trace path

```text
claim_number
    -> claim_id
    -> revision_id (for policy-term lineage)
    -> claim detail / bridge tables
```

### Example SQL pattern

```sql
SELECT c.claim_id, c.claim_number, c.revision_id, c.policy_number
FROM v_claims c
WHERE c.claim_number = 'YOUR_CLAIM_NUMBER';
```

Then fan out to related objects using `claim_id`:

```sql
SELECT *
FROM v_claims_contacts
WHERE claim_id = 'YOUR_CLAIM_ID';
```

## API view

The API layer is often the best place to confirm the resource relationships that exist between entities. The SDK API wrappers and endpoint modules are especially useful because they document the request parameters and the identifiers that the call expects or returns.

For example, the SDK shows:

- `policies.retrieve_policy(policy_number=..., policy_id=...)`
- `policies.retrieve_policy_ids(policy_number=...)` returning `revision_id` and `primary_property_id`
- `claims.get_claim(claim_id=..., claim_number=...)`
- `contacts.get_contact(contact_id=...)`
- `contacts.new_contact(...)` returning a generated `contact_id`

This is important because it gives a direct map of the object identity chain:

- policy lookup by number or id
- revision and property ids returned from a policy lookup
- claim lookup by claim id or claim number
- claim-contact and claim-payment lookups by `claim_id`
- contact lookup by `contact_id`

In other words, the SDK API docs are not just endpoint usage docs; they are a practical relationship guide for how IDs flow between resources.

## CSV view

In the raw CSV layer, the base entity is often the best starting point:

- `revisions.csv` contains `policy_number` and a revision id pattern
- `claims.csv` contains claim-level fields and `policy_number` / `revision_id` linkage
- bridge-style CSV files or relationship files connect those IDs across entities

This means a record trace often begins by finding the stable identifier in the CSV export, then matching that identifier to the logical/reporting object.

The CSV layer is usually more fragmented than the API layer. The raw export often requires several files to reconstruct one full object, because the data is split by concern rather than embedded as nested arrays. For example, a single claim can involve:

- `claims.csv`
- `claims_contacts.csv`
- `claims_payments.csv`
- `claims_payments_item_amounts.csv`
- `claims_perils.csv`
- `claims_changes.csv`
- and related child tables such as `addresses.csv` or `phones.csv`

So the CSV pathway is not necessarily “one file per record”; it is often “one master file plus several child/bridge files that have to be chained together.”

## Cross-domain API pattern across all resource modules

The same relationship pattern is visible across the entire BriteCore SDK, not only in claims, contacts, and policies.

The v2 API modules consistently follow a contract like:

- `create_*` for new objects
- `get_*` / `retrieve_*` for one record by id or stable key
- `list_*` / `search_*` for parent/quote/relationship collections
- `update_*` / `delete_*` for mutation
- relationship-specific calls such as `get_claim_contacts`, `list_vehicles_for_quote`, `list_drivers_for_quote`, and `retrieve_active_file_objects`

Examples from the SDK show the pattern clearly:

- `files.py` uses `generate_download_url(file_id=...)` and `retrieve_active_file_objects(date_added_from=..., label=...)`
- `drivers.py` uses `create_driver(driver)`, `get_driver(id)`, `list_drivers_for_quote(quote_id)`, `update_driver(driver)`
- `vehicles.py` uses `create_vehicle(vehicle)`, `get_vehicle(id)`, `list_vehicles_for_quote(quote_id)`
- `claims.py` uses `get_claim(claim_id=...)`, `get_claim_contacts(claim_id=...)`, `get_claim_payments(claim_id=...)`
- `policies.py` uses `retrieve_policy(policy_number=...)`, `retrieve_policy_ids(policy_number=...)`, `retrieve_policy_terms(...)`

In practice, this means almost every domain can be traced by following a stable identifier to the object id, then to the list of child or related records, then to the next relationship-specific API call.

## API view

The API usually exposes nested payloads rather than a flat row model. For example, a contact or claim record may return:

- object-level fields
- nested arrays such as `addresses`, `phones`, `roles`
- related entity collections rather than a single flat row

This is helpful when you want the current object structure without having to reconstruct every relationship manually, but it is not necessarily the same shape as the raw CSV or the report-view output.

## Recommended trace order

The most reliable general approach is:

1. identify the human-facing key (`policy_number`, `claim_number`, `contact_id`, etc.)
2. find the stable internal id (`revision_id`, `claim_id`, `contact_id`)
3. use that internal id to fan out to related records in the report layer or API
4. compare the raw CSV, API object, and SQL/report view to understand how each layer has shaped the data

## Short rule of thumb

- For policy work, start with `policy_number` and then follow `revision_id`
- For claim work, start with `claim_number` and then follow `claim_id` and `revision_id`
- For contact work, start with `contact_id`, then join to related bridge tables or nested API payloads

This is the practical version of the CSV/API/SQL mapping model and is the safest way to trace a record through the ecosystem.
