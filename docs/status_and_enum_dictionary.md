# Status and Enum Dictionary

This page captures the most important status, enum, and flag fields that appear repeatedly across BriteCore data.

The goal is to reduce the chance of misreading the same concept under different names across CSV, API, and SQL/report layers.

## Why this matters

The same business state can appear in several ways:

- as a raw CSV field like `policyStatus` or `status`
- as an API field like `status`, `policy_status`, or `revision_state`
- as a SQL/report field such as `claim_status`, `policy_status`, or `deleted`

These fields often have different names across layers, but the underlying semantics are very similar.

## Core status families

### Policy and revision status

Common concepts:

- active / inactive
- in force / canceled / expired / rewritten
- renewal status
- revision state and term state

Typical field names:

- CSV: `policyStatus`, `renewalStatus`, `cancelDate`, `newOrRenewalTerm`
- API: `status`, `policy_status`, `renewal_status`, `cancellation` data
- SQL/report: `policy_status`, `revision_state`, `term_effective_date`, `term_expiration_date`

Interpretation rule:

- `policy_status` describes the policy lifecycle state
- `revision_state` describes the revision or term workflow state
- `cancelDate` and cancellation reason are usually historical/operational facts, not the same as a simple current-state status flag

### Claim status

Typical field names:

- CSV: `status`
- API: `status`
- SQL/report: `claim_status`, `claim_active_flag`

Interpretation rule:

- `claim_status` is usually the operational lifecycle state
- `claim_active_flag` is a boolean-ish signal used for current-versus-historical filtering
- status text may include historical or conversion states, so do not assume a narrow modern enum without checking sample data

### Contact status

Typical field names:

- CSV: `active`, `terminated`, `terminationReason`, `flag`
- API: `is_agent`, `terminated`, `termination_reason`, `system_tags`
- SQL/report: `deleted`, `agent_terminated`, `termination_reason`, `roles`

Interpretation rule:

- `deleted` is often a soft-deletion or inactive flag, not always the same as `terminated`
- `terminated` and `agent_terminated` often reflect a business status, not a system-delete state
- `roles` may carry a large amount of semantic status/context beyond a simple boolean flag

## Common boolean and flag patterns

These fields recur often and are usually easy to misread:

- `deleted` / `is_deleted` / `deleted_flag`
- `active` / `is_active` / `active_flag`
- `terminated` / `is_terminated`
- `rewritten`
- `historical`
- `superuser_flag`
- `terms_conditions_accepted`
- `edelivery_enabled`
- `follow_agency_quoting_restriction`

### Rule of thumb

- `deleted` usually means a soft-delete or inactive record in report logic
- `active` usually means currently active in business logic
- `terminated` usually means business termination, not necessarily database deletion
- when a table is historical or reporting-oriented, the flag may mean “currently visible in this grain” rather than “this record has been permanently removed”

## Role and tags vocabulary

### Role fields

Common examples:

- `roles`
- `contact roles`
- `Named Insured`
- `Mortgagee`
- `Claimant`
- `Agent`
- `Agency`

Interpretation rule:

- role data is often denormalized into text in the report layer
- the raw relationship tables may hold the actual role assignments separately
- the report view may store these as comma-delimited strings rather than normalized rows

### System tags

Common examples:

- `systemTags`
- `policySystemTags`
- `claimSystemTags`
- `questionsSystemTags`

Interpretation rule:

- these often behave like a free-form metadata bag
- they are useful for operational filtering but not always clean enough for strict report logic
- do not treat them as canonical master data unless the schema or API docs explicitly say so

## Date and lifecycle semantics

### Date fields that matter most

- `dateAdded` / `date_added`
- `dateUpdated` / `date_updated`
- `effectiveDate` / `term_effective_date`
- `expirationDate` / `term_expiration_date`
- `cancelDate`
- `dateReported`
- `lossDate`
- `commitDate` / `commitDateTime`

Interpretation rule:

- `dateAdded` and `dateUpdated` are operational metadata
- `effectiveDate` and `expirationDate` are term or coverage lifecycle dates
- `cancelDate` and `terminationReason` are often historical lifecycle indicators
- `lossDate` and `dateReported` are claim event dates, not necessarily policy-term dates

## Common status confusion patterns

### Policy and revision confusion

A common mistake is to interpret the latest policy row as the same thing as the current revision or the current term.

The correct mental model is:

- policy number identifies the policy
- revision captures the term or change iteration
- a policy can have many revisions over time

### Claim confusion

A claim can have many lifecycle and status changes, and the claim table may carry both operational status and historical content. Do not assume a single status field tells the full story.

### Contact confusion

A contact may be active in one context and inactive or terminated in another. A single contact record can carry both role and status information applied across multiple policy or claim relationships.

## Recommended reporting rule

When you are writing business logic, use this sequence:

1. identify the business state you want
2. determine whether it is a master-object status or a relationship status
3. determine whether it is historical or currently active
4. validate against both the raw CSV and the report view before assuming semantics

## Short reference list

| Concept | Typical naming patterns | Meaning |
|---|---|---|
| Active | `active`, `is_active`, `active_flag` | currently active / enabled |
| Deleted | `deleted`, `is_deleted` | soft-delete or inactive status |
| Terminated | `terminated`, `agent_terminated` | business or user termination status |
| Cancelled | `cancelDate`, `cancellation_reason` | cancellation lifecycle event |
| Revision state | `revision_state`, `revisionDate` | term/change lifecycle |
| Policy status | `policy_status`, `policyStatus` | policy lifecycle state |
| Claim status | `claim_status`, `status` | claim operational state |
| Role | `roles`, `role_name` | relationship / assigned business role |
| Tag | `systemTags`, `claimSystemTags` | metadata or categorization label |
| Historical | `historical` | record retained for prior-state context |

## Good operational rule

If a field name looks like a status or tag, always check:

- Is it master-record status or relationship status?
- Is it current-state or historical-state?
- Is it a boolean flag, a text enum, or a denormalized text field?

That question will usually resolve most of the ambiguity quickly.
