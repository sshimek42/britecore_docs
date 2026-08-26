# Contact Domain Guide

This page is the practical guide for working with contact records across raw CSV exports, the live API, and the reporting layer.

## What a contact is

A contact is the master person or organization record used across policy, claim, agency, billing, and servicing workflows.

A contact can be linked to:

- addresses
- phones
- emails
- roles
- claims
- policy revisions
- agencies
- user/auth metadata

## Core identity

The most important keys are:

- `contact_id` — canonical internal identity
- `contactId` in CSV exports
- `id` in API objects
- `contact_name` and `contact_type` for the readable identity

Typical identity pattern:

```text
contactId / id / contact_id -> contact master row
```

## Best starting tables

### CSV

- `contacts.csv` — contact master row
- `addresses.csv` — address detail rows keyed on `contactId`
- `phones.csv` — phone detail rows keyed on `contactId`
- `emails.csv` — email detail rows keyed on `contactId`
- agency or role lookup files when present

### API

- `contacts.get_contact(contact_id=...)`
- `contacts.retrieve_contact(...)`
- `contacts.list_contacts(...)`
- nested `addresses`, `phones`, `emails`, and `roles` collections in the payload

### SQL/report

- `v_contacts` — master contact record
- `v_addresses` — address child rows
- `v_phones` — phone child rows
- `v_emails` — email child rows
- `v_claims_contacts` and `v_revisions_contacts` — relationship rows that link contacts to claims/revisions

## Typical contact trace pattern

### Find the contact master row

```sql
SELECT c.contact_id,
       c.contact_name,
       c.contact_type,
       c.roles,
       c.primary_email,
       c.primary_phone
FROM v_contacts c
WHERE c.contact_id = 'YOUR_CONTACT_ID';
```

### Pull the contact’s address and communication data

```sql
SELECT c.contact_id,
       c.contact_name,
       a.address_line_1,
       a.address_city,
       a.address_state,
       a.address_zip,
       p.phone,
       p.phone_type,
       e.email,
       e.email_type
FROM v_contacts c
LEFT JOIN v_addresses a
  ON a.contact_id = c.contact_id
LEFT JOIN v_phones p
  ON p.contact_id = c.contact_id
LEFT JOIN v_emails e
  ON e.contact_id = c.contact_id
WHERE c.contact_id = 'YOUR_CONTACT_ID';
```

### Find the claims or revisions the contact is attached to

```sql
SELECT cc.claim_id, c.claim_number, c.claim_status
FROM v_claims_contacts cc
LEFT JOIN v_claims c
  ON c.claim_id = cc.claim_id
WHERE cc.contact_id = 'YOUR_CONTACT_ID';
```

```sql
SELECT rc.revision_id, r.policy_number
FROM v_revisions_contacts rc
LEFT JOIN v_revisions r
  ON r.revision_id = rc.revision_id
WHERE rc.contact_id = 'YOUR_CONTACT_ID';
```

## CSV model

The raw CSV layer usually splits contact data into:

- `contacts.csv` for the core record
- `addresses.csv` for addresses
- `phones.csv` for phones
- `emails.csv` for emails
- other role or association files where needed

This is why a “full contact” reconstruction usually requires more than one CSV file.

## API model

The API usually returns a single nested object for the contact, with child arrays.

A typical object might contain:

- `id`
- `name`
- `type`
- `addresses[]`
- `phones[]`
- `emails[]`
- `roles[]`

This is often the easiest way to read the current structural shape before flattening for SQL/report use.

## Report model

The report layer is usually flattened into a denormalized master row:

- `v_contacts` is the contact master
- communication fields are often flattened into core columns
- `roles` may be serialized text rather than a normalized child table

This is useful for reporting, but not always the same as the raw or API object model.

## Key fields to watch

High-value fields include:

- `contact_id`
- `contact_name`
- `contact_type`
- `roles`
- `primary_email`
- `primary_phone`
- `agency_number`
- `producer_number`
- `vendor_number`
- `deleted`, `terminated`, `termination_reason`
- timestamps such as `date_added` and `date_updated`

## Common gotchas

### 1. A contact can have multiple addresses and phone numbers

Do not assume one row per phone or address. The child tables are usually one row per communication record.

### 2. `roles` may be denormalized text

It may look like a single text field rather than a clean child table, especially in report views.

### 3. Use `contact_id` for the relationship join, not the display name

The human-readable name is not a reliable join key in the relationship tables.

## Recommended working model

For contact work, the pattern is usually:

1. find the contact by `contact_id` or name lookup
2. confirm the master row in `v_contacts` or `contacts.csv`
3. pull address/phone/email child rows from their respective detail tables
4. fan out to claim or revision relationships via `v_claims_contacts` or `v_revisions_contacts`

That keeps the relationship grain under control and avoids confusing the master contact with the child/contact-link records.

## Contact quick reference

```text
contact_id -> master contact -> addresses / phones / emails / roles
contact_id -> claim relationships / revision relationships
```

This is the working contact lineage model across the ecosystem.
