# Reporting Playbook

This playbook captures the repeatable patterns used to define, validate, and deliver BriteCore reporting logic across the local stack.

## Goal

The goal is to produce reliable reporting logic that is:

- aligned to the actual business question
- anchored on the correct grain
- joined only as needed
- validated against the correct source system
- documented in a way that survives team turnover

## Step 1: Define the business question

A strong report definition should answer these questions:

- What is the measure or fact being analyzed?
- Which entity is the primary subject: policy, claim, premium, contact, account, revision, or file?
- What is the expected row grain?
- What time horizon matters?
- Are we answering a status query, a historical query, or a reconciliation query?

Without a clear question and grain, most reporting issues come from downstream join choices rather than true logic errors.

## Step 2: Pick the anchor object

The anchor object should match the core subject of the question.

### Common anchors

- Policy reporting: `m_inforce_policies`, `v_policies`, or related revision references
- Claims: `v_claims` with claim-date and claim-item context
- Premiums: `m_premium_terms`, `m_premium_transactions`, or premium summary tables
- Contacts: `v_contacts`, `v_insureds`, or `v_claims_contacts`
- Audit/lineage: `v_revisions` and adjacent revision tables

A good anchor is one that:

- matches the natural grain of the report
- contains the fact columns or the most stable primary keys
- is easy to explain to a business stakeholder

## Step 3: Discovery before join logic

Before writing joins, determine:

- which views are facts
- which views are dimensions or bridges
- which views describe business context
- whether the report is based on a logical SQL layer or raw export lineage

The best discovery source is usually:

```sql
SELECT *
FROM v_logical_catalog;
```

Use this to identify:

- candidate fact views
- lookups and bridge tables
- field descriptions and names
- likely join keys

## Step 4: Choose a join strategy

### Start simple

Join only the views needed for the answer.

A safe pattern is:

- anchor on the business entity or fact table
- add one dimension or lookup at a time
- stop when the output is semantically complete

### Avoid join-heavy patterns by default

Large or repetitive bridge tables can inflate row counts quickly. It is common for a report to look correct in principle but fail in practice because the join fan-out is too large.

## Step 5: Validate grain and fan-out

Every report should answer these questions:

- Does each row represent one policy, one claim, one premium record, or some other unit?
- Are there duplicate rows after joins?
- Are we inflating counts because of many-to-many relationships?
- Are dates or effective periods being applied correctly?

Use these checks:

- row counts before and after each join
- distinct key counts for the anchor entity
- check for duplicate keys on the expected grain
- compare to raw export relationships when the logical layer is unclear

## Step 6: Validate with raw export context when needed

Use `britecore-csv-loader` when:

- the SQL view relationships are not obvious
- row counts or joins do not line up with business expectations
- the underlying data is derived from exported CSVs and revision snapshots
- you need to confirm whether the logical layer matches the raw data model

This is especially valuable for claims, revisions, and policy lifecycle scenarios where relationship intensity is high.

## Domain starter patterns

### Policy and in-force reporting

Anchor to:

- `m_inforce_policies`
- `v_policy_types`
- `v_revisions`
- `v_insureds`
- `v_properties`

### Claims reporting

Anchor to:

- `v_claims`
- `v_claims_dates`
- `v_claims_perils`
- `v_claims_contacts`
- `v_claim_payments`
- `v_claim_items`

### Premium reporting

Anchor to:

- `m_premium_terms`
- `m_premium_transactions`
- `v_account_history`
- `v_return_premium`
- `v_policy_types`

### Audit and lineage reporting

Anchor to:

- `v_revisions`
- `v_revision_items`
- `v_claim_change_log`
- `v_policy_change_log`

## Quality checklist

Before finalizing a report, verify:

- The business question is clearly stated.
- The anchor object is explicit.
- The row grain is known.
- The required fact and dimension tables are identified.
- Joins are constrained to necessary enrichment.
- Duplicate rows or fan-out are checked.
- The final query has a clear documented source-of-truth narrative.

## Example report assembly pattern

```text
Business question:
"What are the active policies, by policy type and insured, for all policies with a claim in 2025?"

1. Anchor on active policy facts.
2. Join policy type and insured dimensions.
3. Add claim event facts via claim relationship tables.
4. Apply the claim date filter.
5. Validate row count and unique policy grain.
6. Document the join path and definitions.
```

## Recommended source documents

- `britecore_mcp` logical catalog and report docs
- `docs/reporting/logical_layer.md`
- `docs/reporting/join_starters.md`
- `britecore-csv-loader` raw relationship docs when validation is needed

## Definition of done

A reporting solution is ready when:

- stakeholders can explain the question and output grain
- the key views are documented
- join logic is intentional, not accidental
- assumptions have been validated against facts or raw relationships
- the logic is maintainable by another analyst without reverse-engineering the whole query
