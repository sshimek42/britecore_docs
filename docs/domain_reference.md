# Domain Reference

This page is the high-level map for the main BriteCore domains. It is meant to help you decide which dataset, API, or report view to use for a given domain without re-reading the whole ecosystem.

## Domain map

### Claims

Primary concern:

- claim lifecycle
- claim status
- claim parties
- claim payments and item detail
- loss and catastrophe context

Best starting points:

- `claims.csv` or `v_claims`
- `claims_contacts.csv` or `v_claims_contacts`
- `claims_payments.csv` or `v_claim_payments`
- `claim_items.csv` or `v_claim_items`

Core join keys:

- `claim_id`, `claim_number`, `revision_id`, `policy_number`

Typical lineage:

```text
claim_number -> claim_id -> claim_contacts / claim_payments / claim_items
```

### Policies and revisions

Primary concern:

- policy identity
- term lifecycle
- revision history
- billing and policy status

Best starting points:

- `policies.csv` or policy master records
- `policy_terms.csv` or `v_revisions`
- `revisions.csv`

Core join keys:

- `policy_number`, `policy_id`, `revision_id`, `policy_term_id`

Typical lineage:

```text
policy_number -> revision_id -> claims / properties / contacts / agencies
```

### Contacts

Primary concern:

- person or organization identity
- addresses, phones, emails
- agency and user metadata
- role assignments and claim/revision participation

Best starting points:

- `contacts.csv` or `v_contacts`
- `addresses.csv` or `v_addresses`
- `phones.csv` or `v_phones`
- `emails.csv` or `v_emails`

Core join keys:

- `contact_id`, `contact_number`, `agency_number`, `producer_number`

Typical lineage:

```text
contact_id -> addresses / phones / emails / roles
contact_id -> claim contacts / revision contacts
```

### Properties and risks

Primary concern:

- insured property or risk records
- address and location info
- risk details and inspection metadata
- association to revisions and mortgagees

Best starting points:

- `properties.csv` or `v_properties`
- `mortgagees.csv` or `v_mortgagees`
- `property_items.csv` or `v_property_items`

Core join keys:

- `property_id`, `revision_id`, `property_name`

Typical lineage:

```text
revision_id -> property_id -> mortgagees / property items
```

### Payments and billing

Primary concern:

- policy billing and transaction history
- claim payments and item splits
- premium, accounting, and invoice context

Best starting points:

- `policy_payments.csv`
- `claim_payments.csv` or `v_claim_payments`
- `claims_payments_item_amounts.csv` or `v_claim_payments_item_amounts`
- `account_history.csv`

Core join keys:

- `policy_id`, `revision_id`, `claim_id`, `claim_payment_id`

Typical lineage:

```text
claim_id -> claim_payment_id -> item amounts
policy_id -> payment records / billing schedule
```

### Files and other attached objects

Primary concern:

- document and file objects
- generated downloads and active file objects
- file metadata tied to policies or claims

Best starting points:

- `files.py` API wrappers
- `generate_download_url(file_id=...)`
- `retrieve_active_file_objects(...)`

Typical lineage:

```text
file_id -> download URL / active file metadata
```

### Vehicles, drivers, and related risk data

Primary concern:

- vehicle and driver records attached to quote or policy workflows
- quote-scoped object linkage

Best starting points:

- `drivers.py`
- `vehicles.py`
- `list_drivers_for_quote(quote_id=...)`
- `list_vehicles_for_quote(quote_id=...)`

Typical lineage:

```text
quote_id -> vehicle_id / driver_id -> object detail
```

## How to choose the right layer per domain

### Use CSV when:

- you want the raw data shape
- you are validating a source export
- you need point-in-time, offline access
- you are reconstructing lineage from child/bridge tables

### Use API when:

- you need a live object state
- the nested object structure is useful
- you want child collections without reconstructing separate CSVs

### Use SQL/report views when:

- you need report-ready flattened data
- you want a simpler working answer for a business question
- you are in an analysis workflow that should minimize raw join complexity

## Domain decision cheat sheet

| Domain | Best starting point | Best follow-on detail | Common join keys |
|---|---|---|---|
| Claims | `v_claims` / `claims.csv` | `v_claims_contacts`, `v_claim_payments` | `claim_id`, `revision_id`, `policy_number` |
| Policies / revisions | `v_revisions` | `v_claims`, `v_properties`, `v_revisions_contacts` | `revision_id`, `policy_number`, `policy_id` |
| Contacts | `v_contacts` | `v_addresses`, `v_phones`, `v_emails` | `contact_id` |
| Properties | `v_properties` | `v_mortgagees`, `v_property_items` | `property_id`, `revision_id` |
| Payments | `v_claim_payments` | payment item amounts / billing tables | `claim_id`, `claim_payment_id`, `policy_id` |
| Files | API file wrappers | active file object metadata | `file_id` |
| Vehicles / drivers | API modules | quote-linked list endpoints | `quote_id`, `vehicle_id`, `driver_id` |

## Domain-level quick rule

If you know the domain, start with the base entity table and then move outward using the ids that the ecosystem consistently reuses:

- policy → revision → claim/property/contact
- claim → claim contacts/payments/items
- contact → addresses/phones/emails/roles
- property → revision + mortgagees + property items

That is the fastest route to reliable cross-layer reconstruction.
