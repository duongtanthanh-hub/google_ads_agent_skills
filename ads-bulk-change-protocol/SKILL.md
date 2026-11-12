---
name: ads-bulk-change-protocol
description: >-
  Plans and checks bulk Google Ads API writes (more than 100 entities or more
  than 10,000 operations): exact-payload validate_only dry run, choice between
  BatchJobService and chunked synchronous mutates within API limits,
  partial-failure handling and reconciliation of every failed operation,
  staged waves, failure-rate caps, quota forecast, rollback snapshot, labelling
  of agent-created entities and post-wave checks (EXE-01…06, EXE-08, STR-02,
  STR-04, OPS-02…06). Returns a verdict, the T3 approval package and a wave
  plan; read-only. Use before any large create, edit, pause or listing-group
  job, and between waves. Don't use for a single small change (use
  ads-change-guardrails), for general API read/write habits (use
  ads-api-hygiene), or for judging results after the change (use
  ads-post-change-check).
---

# Bulk change protocol

Large API jobs fail differently from single edits: a 3% failure rate on 20
million operations is about 600,000 broken changes, and a quota-hungry job can
stop the reports everyone else relies on. This skill plans the job so it is
validated, staged, reconciled and reversible, and checks each wave before the
next one. It never writes; the executing agent runs each wave after approval.

## When to use

-   **Before a bulk job**: > 100 entities (EXE-05 T3) or > 10,000 operations
    (EXE-02 per-request limit), or any cross-account job.
-   **Between waves**: to decide go / pause / halt from the wave's results.
-   **After an incident**: a job with many failures or a quota outage, to plan
    the clean-up.

Not for: one change to ≤ 100 entities
([ads-change-guardrails](../ads-change-guardrails/SKILL.md)), API efficiency in
general ([ads-api-hygiene](../ads-api-hygiene/SKILL.md)), performance after the
change ([ads-post-change-check](../ads-post-change-check/SKILL.md)).

## Inputs

Input                        | Required      | Notes
---------------------------- | ------------- | -----
Job spec                     | yes           | Accounts, entity types, operation types and counts, purpose, requester, thread
Payload (or generator)       | yes           | The exact operations, so the dry run tests what will run
Access level and daily quota | yes           | Of the Google Cloud project the job uses; plus scheduled jobs' usage
Wave results                 | between waves | Per-operation results, error codes, read-back
`../_shared/client_rules.md` | yes           | Labels, naming, protected terms, approvers

## Workflow

1.  **Load the rules.** Read `../_shared/thresholds.md`,
    `../_shared/client_rules.md` and `../_shared/output_contract.md`. Use only
    those numbers and the OPS IDs; never invent thresholds.
2.  **Size the job.** Count entities, operations by type, accounts, new
    campaigns. Confirm it is bulk; if not, use ads-change-guardrails.
3.  **Check what the job does, not just how.** Run
    [ads-change-guardrails](../ads-change-guardrails/SKILL.md) on each distinct
    change type (one representative entity plus the aggregate): CHG, STR-01…03,
    STE-04. A bulk job of rule-breaking changes is still rule-breaking.
4.  **Choose the write method** with
    [01_method_and_limits.md](references/01_method_and_limits.md): chunked
    `Mutate` (≤ 10,000 operations per request, EXE-02) or `BatchJobService`.
    Note atomic groups and ordering.
5.  **Plan the dry run** of the exact payload with `validate_only` (EXE-01),
    wave by wave, and count its cost in the quota forecast
    ([02_dry_run_waves_quota.md](references/02_dry_run_waves_quota.md)).
6.  **Forecast quota** (EXE-06): operations per day for dry run + writes +
    read-back + retries, against the access level, keeping headroom for
    scheduled reports.
7.  **Plan waves** (EXE-04) and **new-campaign pacing** (STR-02): first wave ≤ 1
    account or ≤ 10% of entities; ≥ 24 h between waves.
8.  **Plan safety nets** with
    [03_failures_reconciliation_rollback.md](references/03_failures_reconciliation_rollback.md):
    rollback snapshot (OPS-03), no removals without one (OPS-02), retry policy
    (OPS-04), reconciliation ledger (EXE-02), read-back (OPS-05), labels and
    identity (STR-04, OPS-06), post-wave checks (EXE-08).
9.  **Verdict and tier.** Bulk jobs are T3 (EXE-05). Reply in the
    output-contract format, then the wave plan, then numbered options (e.g.
    smaller first wave, consolidate instead of creating). Close with "I'll start
    wave 1 only after explicit T3 approval from <admin> in this thread."
10. **Between waves**, apply the gate in
    [02_dry_run_waves_quota.md](references/02_dry_run_waves_quota.md): EXE-03
    failure rate, reconciliation complete, read-back match, D+1 health (OPS-08).
    Go, pause or halt; never "go" with unreconciled failures.

## Account selection and fallback (standalone)

As in the public `google-ads-api-account-diagnostics` skill: check the MCP tools
(`search`, `list_accessible_customers`, `get_resource_metadata`); find enabled
non-manager accounts with the `customer_client` query in
[queries.md](references/queries.md). Without the tools, return the plan and
queries for manual use, mark live checks NOT_ASSESSED, and point to
`google-ads-api-mcp-setup`.

## Gotchas

-   **Partial failure hides failures.** Batch jobs always run with partial
    failure; a "DONE" job can contain thousands of failed operations. Read every
    result.
-   **Some batch-job groups are atomic.** Consecutive listing-group filter
    operations for one asset group, or an asset group plus its assets, succeed
    or fail together; one bad operation fails the group.
-   **`validate_only` isn't free.** Plan as if dry-run operations count against
    the daily quota (G; confirm), and forecast them.
-   **Retries can double-apply** non-idempotent creates. Before retrying a
    create, check by name and label whether it already exists.
-   **Don't parallelise writes to the same campaign or budget**; they fail with
    `CONCURRENT_MODIFICATION`. Group by parent; one writer per parent.
-   **Removing is permanent**, and a bulk remove is the costliest mistake.
    Prefer pause (OPS-02).
-   **Asset groups can't carry labels** in API v24; mark them by name, and label
    the parent campaign.
-   **The job's identity matters**: run it under the agent's own login identity
    so `change_event` can separate it from people (OPS-06).

## Sources

Public: Google Ads API developer guide (batch processing, mutate best practices,
partial failure, `validate_only`, temporary IDs, quotas and access levels,
errors such as `TOO_MANY_MUTATE_OPERATIONS` and `CONCURRENT_MODIFICATION`),
Google Ads Developer Blog (batch job limits), Google Ads Help (removed
campaigns, labels). G and W labels in `thresholds.md` mark guidance being
confirmed and working proposals.
