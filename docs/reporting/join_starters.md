# Join Starters

This page mirrors the practical reporting guidance developed in `britecore_mcp` and is intended as a cross-project reference.

## Quick selection rules

- Start with the object that matches the business question.
- Define grain before joining.
- Add only the enrichment views you need.
- Validate row-count impact when joining bridges and multi-row event tables.

## Domain starters

### Policy / in-force
- anchor: `m_inforce_policies`
- common joins: `v_policy_types`, `v_revisions`, `v_insureds`, `v_properties`

### Premium
- anchor: `m_premium_terms`, `m_premium_transactions`, or `m_premium_mtd_ytd`
- common joins: `v_revisions`, `v_policy_types`, `v_insureds`, `v_account_history`, `v_return_premium`

### Claims
- anchor: `v_claims`
- common joins: `v_claims_dates`, `v_claims_perils`, `v_claims_contacts`, `v_claim_payments`, `v_claim_items`, `v_claims_disputes`

### Contact / insured
- anchor: `v_contacts` or `v_insureds`
- common joins: `v_addresses`, `v_contact_relationships`, `v_claims_contacts`, `v_roles`, `v_credit_reports`

### Commission
- anchor: `v_commission_details` or `v_commission_payments`
- common joins: `v_contacts`, `v_revisions`, and each other via `commission_payment_id`

### Audit / lineage
- anchor: `v_revisions`, `v_revision_items`, `v_claim_change_log`, or `v_policy_change_log`
- common joins: `v_contacts`, `v_policy_types`, `v_claims`, `v_files`

## Canonical deeper reference

Detailed, evolving guidance lives in:
- `C:\PythonProjects\BriteCore\britecore_mcp\scripts\report_join_starters.md`

