# Field Mapping Cheat Sheet

This page is the practical side-by-side reference for the most common BriteCore entities. It is meant to reduce the friction of translating between:

- raw CSV exports
- API objects and nested JSON payloads
- SQL/report views such as `v_*`

The goal is not to list every possible field in every system. The goal is to show the common identifiers and the highest-value fields that are used for joins, lookups, and the first pass of reconstruction.

## How to read this sheet

The same record often appears in three shapes:

1. CSV: flat, raw, and split by concern
2. API: nested object/payload with child arrays
3. SQL/report: flattened, denormalized view

A very reliable rule is that the API and SQL/report layers usually match each other much more closely than either matches the raw CSV naming convention. The CSV layer tends to preserve legacy or export-style names such as `contactId`, `claimNumber`, and `policyNumber`, while the API and report layers usually normalize to `contact_id`, `claim_number`, and `policy_number`.

The key is usually the same across all three:

- base entity id
- business key or number
- parent/child linkage id
- date or status fields for the lifecycle

## Core identifier patterns

These are the most important recurring patterns to remember:

- `contactId` / `contact_id` / `id` → contact identity
- `claimId` / `claim_id` / `id` → claim identity
- `revisionId` / `revision_id` → policy-term lineage anchor
- `policyId` / `policy_id` / `policy_number` → policy identity and human-readable lookup
- `propertyId` / `property_id` → property/risk identity
- `contactId` + `claimId` in bridge tables → claim-to-contact relationship rows
- `revisionId` + `contactId` in bridge tables → revision-to-contact relationship rows

## Contacts

### Most important fields

| Concern | CSV field names | API object pattern | SQL/report pattern |
|---|---|---|---|
| Contact id | `contactId` | `id` | `contact_id` |
| Name | `name` | `name` | `contact_name` |
| Type | `type` | `type` | `contact_type` |
| Agency fields | `agencyNumber`, `producerNumber`, `vendorNumber` | `agency_number`, `producer_number`, `vendor_number` | `agency_number`, `producer_number`, `vendor_number` |
| Email | `noticeEmailId`, `confirmationEmail`, `claimActivityEmailId` | `emails[]`, `primary_email_id` | `primary_email`, `user_email` |
| Phone | `phones` are in a separate file; raw CSV `phones.csv` | `phones[]` | `primary_phone`, `cell_phone`, `home_phone`, `work_phone` |
| Address | `addresses` are split across `addresses.csv` | `addresses[]` | `address_line_1`, `address_city`, `address_state`, `address_zip` |
| Roles | `systemTags`, `position` | `roles[]` with `name` values | `roles` |
| Status | `active`, `terminated`, `terminationReason` | `is_agent`, `terminated`, `termination_reason` | `deleted`, `agent_terminated`, `termination_reason` |
| Timestamps | `dateAdded`, `dateUpdated` | `source_update_date`, `date_added` / `date_updated` variants | `date_added`, `date_updated` |

### Typical contact reconstruction flow

From raw CSV:

- start with `contacts.csv` for the core record
- then join to `addresses.csv` on `contactId`
- then join to `phones.csv` on `contactId`
- then join to `emails.csv` on `contactId`
- relationship records may come from bridge or lookup files

From API:

- one call to `contacts.get_contact(contact_id=...)`
- nested arrays such as `addresses`, `phones`, `emails`, `roles`

From SQL/report:

- base table: `v_contacts`
- one row per contact (`contact_id`)
- flattened communication and role fields

### Example identity mapping

```text
CSV: contactId = 00025e4c-af2e-4a1e-bdc6-3808ae83e7ce
API: id = 00025e4c-af2e-4a1e-bdc6-3808ae83e7ce
SQL: contact_id = 00025e4c-af2e-4a1e-bdc6-3808ae83e7ce
```

## Claims

### Most important fields

| Concern | CSV field names | API object pattern | SQL/report pattern |
|---|---|---|---|
| Claim id | `id` | `id` | `claim_id` |
| Claim number | `claimNumber` | `claim_number` | `claim_number` |
| Policy linkage | `policyId`, `policyNumber`, `revisionId` | `policy_id`, `policy_number`, `revision_id` | `policy_number`, `revision_id`, `policy_type_id` |
| Status | `status` | `status` | `claim_status` |
| Type | `claimType` | `claim_type` | `claim_type` |
| Loss date | `lossDate` | `loss_date` | `loss_date` |
| Date reported | `dateReported` | `date_reported` | `date_reported` |
| Location | `lossAddressLine1`, `lossAddressCity`, `lossAddressState`, `lossAddressZip` | `loss_address` or nested location object | `loss_location_address_1`, `loss_location_address_city`, `loss_location_address_state` |
| Catastrophe | `catId` | `catastrophe_id` | `catastrophe_id`, `cat_code`, `cat_pcs` |
| Description | `description` | `description` | `description` |
| Related records | `claims_contacts.csv`, `claims_payments.csv`, `claims_perils.csv` | `claim_contacts`, `claim_payments`, `claim_items` endpoints | `v_claims_contacts`, `v_claim_payments`, `v_claim_items` |

### Typical claim reconstruction flow

From raw CSV:

- `claims.csv` gives the claim master record
- `claims_contacts.csv` gives the people and parties on the claim
- `claims_payments.csv` gives payment records
- `claims_payments_item_amounts.csv` gives per-item payment detail
- `claims_perils.csv` gives coverage/peril detail
- `claims_changes.csv` gives the lifecycle or change trail

From API:

- `claims.get_claim(claim_id=...)` or `claims.get_claim(claim_number=...)`
- then relationship calls like `get_claim_contacts`, `get_claim_payments`

From SQL/report:

- base table: `v_claims`
- detail views: `v_claims_contacts`, `v_claim_payments`, `v_claim_items`, `v_claims_dates`, `v_claims_perils`

## Policies and revisions

### Most important fields

| Concern | CSV field names | API object pattern | SQL/report pattern |
|---|---|---|---|
| Policy number | `policyNumber` | `policy_number` | `policy_number` |
| Policy id | `policyId` | `id` or `policy_id` | `policy_id` |
| Revision id | `revisionId` | `revision_id` | `revision_id` |
| Policy term id | `policyTermId` | `policy_term_id` | `policy_term_id` |
| Effective date | `effectiveDate` | `effective_date` | `term_effective_date` |
| Expiration date | `expirationDate` | `expiration_date` | `term_expiration_date` |
| Status | `policyStatus` | `status` or `policy_status` | `policy_status` |
| Billing | `billingSchedule` | `billing_schedule` | `billing_schedule` |
| Cancellation | `cancelDate`, `policyCancellationReason` | `cancellation` data | `cancel_date`, `policy_cancellation_reason` |

### Typical policy/revision reconstruction flow

From raw CSV:

- `policies.csv` gives the policy master identity
- `policy_terms.csv` gives term-level dates and renewal status
- `revisions.csv` gives revision lifecycle and policy status detail
- related files such as `addresses.csv`, `phones.csv`, `mortgagees.csv`, and `properties.csv` add child context

From API:

- `policies.retrieve_policy(policy_number=...)`
- `policies.retrieve_policy_ids(policy_number=...)` returns the id chain, including `revision_id`
- `policies.retrieve_policy_terms(...)` and `policies.retrieve_revision_details(...)` add term/revision context

From SQL/report:

- base table: `v_revisions`
- additional policy tables and relationship views attach the policy context, contacts, agencies, properties, and claims

### Key lineage rule

This is the central policy lineage pattern:

```text
policy_number
  -> policy_id / revision_id
  -> revision_id -> claims / properties / contacts / agencies
```

## Properties and risks

### Most important fields

| Concern | CSV field names | API object pattern | SQL/report pattern |
|---|---|---|---|
| Property id | `propertyId` | `id` or `property_id` | `property_id` |
| Revision id | `revisionId` | `revision_id` | `revision_id` |
| Address | `riskAddressLine1`, `riskCity`, `riskState`, `riskZip` | `address` / nested property location | `risk_address_line_1`, `risk_city`, `risk_state`, `risk_zip` |
| County | `riskCounty` | `county` or `county_id` | `risk_county` or county-related fields |
| Name | `propertyName` | `name` | `property_name` |
| Inspection | `riskInspectionRequestedDate`, `riskNextInspectionDate` | inspection metadata | inspection or risk-detail fields |
| Location metadata | `riskLatitude`, `riskLongitude` | `latitude`, `longitude` | `risk_latitude`, `risk_longitude` |

### Typical property reconstruction flow

- `properties.csv` gives the master property/risk rows
- `revisionId` ties the property back to the policy revision
- `mortgagees.csv` can add mortgagee/interest records tied to the property and revision

## Payments and monetary records

### Most important fields

| Concern | CSV field names | API object pattern | SQL/report pattern |
|---|---|---|---|
| Payment record id | `id` | `id` | payment id or related payment table |
| Claim linkage | `claimId` | `claim_id` | `claim_id` |
| Policy linkage | `policyId`, `revisionId` | `policy_id`, `revision_id` | `policy_id`, `revision_id` |
| Amount | `amount` / `claimPaymentAmount` | `amount` | payment amount columns |
| Date | `transactionDate` | `transaction_date` | `transaction_date` |
| Status/voids | `voided`, `voidedDate` | `voided`, `voided_date` | `voided`, `voided_date` |

### Typical payment reconstruction flow

- `claims_payments.csv` gives claim-payment records
- `claims_payments_item_amounts.csv` gives the itemized amounts within each payment
- `policy_payments.csv` or similar may hold payment records at the policy layer

## Recommended mapping patterns

When you are mapping a record across layers, use this order:

1. find the base entity row in the CSV export
2. identify the stable id and business key
3. map to the same key in the API payload
4. join to the corresponding SQL/report view by the same id
5. then fan out through bridge tables using the relationship keys

## Short rule of thumb

- CSVs usually give the raw grain and the entity-level key names
- APIs usually give the current nested object plus child collections
- report views usually flatten and join the same data for business reporting

If you keep the base id and parent-child linkage id in mind, almost all of the entity mapping becomes consistent.
