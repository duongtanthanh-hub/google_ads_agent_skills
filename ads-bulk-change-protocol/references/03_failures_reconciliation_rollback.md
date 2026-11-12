# 3. Failures, reconciliation, rollback and labels

## Classify every failed operation (OPS-04)

Class              | Typical error codes (v24)                                                                                                                                                                                                                                        | What to do
------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------
Transient          | `QuotaError.RESOURCE_TEMPORARILY_EXHAUSTED`, `DatabaseError.CONCURRENT_MODIFICATION`, `InternalError.INTERNAL_ERROR`, `InternalError.TRANSIENT_ERROR`, `InternalError.DEADLINE_EXCEEDED`                                                                         | Retry with exponential backoff and jitter, honouring `QuotaErrorDetails.retry_delay` when present; cap attempts per OPS-04
Quota exhausted    | `QuotaError.RESOURCE_EXHAUSTED` ("too many requests"). Read `QuotaErrorDetails`: `rate_scope` `DEVELOPER` = per-project quota (formerly the developer-token quota, e.g. the daily operations limit), `ACCOUNT` = per-account quota; `rate_name` names the bucket | Stop the job and report it in the thread. If it is the daily project quota, resume after the reset with a new forecast; otherwise back off per `retry_delay`. Never loop
Query cost         | `EXCESSIVE_SHORT_TERM_QUERY_RESOURCE_CONSUMPTION`, `EXCESSIVE_LONG_TERM_QUERY_RESOURCE_CONSUMPTION`                                                                                                                                                              | Reads are too heavy; slow down and simplify queries (see ads-api-hygiene)
Permanent: payload | Field, range, format, `RequestError.*`, `DatabaseError.DATA_CONSTRAINT_VIOLATION`, resource limits                                                                                                                                                               | Don't retry. Fix the generator, re-run the dry run, include in a later wave
Permanent: state   | `OperationAccessDeniedError.OPERATION_NOT_PERMITTED_FOR_REMOVED_RESOURCE`, entity already exists, parent missing                                                                                                                                                 | Don't retry. Check the current state; drop or re-plan the operation
Permanent: policy  | Policy violation errors on ads or assets                                                                                                                                                                                                                         | Don't retry or rephrase to get past review. Send to a person
Permanent: access  | `AuthorizationError.*`, `AuthenticationError.*`                                                                                                                                                                                                                  | Stop. Fix access through the documented path; never work around it
Atomic group       | `ASSET_GROUP_LISTING_GROUP_FILTER_TRANSACTION_FAILURE`, `ASSET_GROUP_AND_ASSET_GROUP_ASSET_TRANSACTION_FAILURE`, `CAMPAIGN_AND_CAMPAIGN_ASSET_TRANSACTION_FAILURE`                                                                                               | Find the real error in the same group's other results; fix that; resubmit the whole group

**Before retrying a create**, check whether it already exists (by name and job
label). A create that timed out may have succeeded.

## Reconciliation ledger (EXE-02)

Keep one row per submitted operation: wave, operation index, entity, operation
type, payload hash, result (`OK` / error code / not executed), class, action
(retried, fixed in wave N, written off with reason), final state.

Check        | How to judge
------------ | ------------
Completeness | Succeeded + failed + not executed = submitted, per wave. Any gap → Halt
Not executed | Batch job `executed_operation_count` < `operation_count` (cancelled or stopped) → those operations are "not executed", never assumed applied
Written off  | Each write-off has a reason and is listed in the wave report
Failure rate | EXE-03 uses failed / submitted for the wave, before retries. Report the retry outcome separately

## Read-back (OPS-05)

After each wave, read the live state of changed entities with GAQL (see Q3 in
[queries.md](queries.md)) and compare with the payload. Sample size per OPS-05;
always include every entity that had a failure or retry. A mismatch with no
explanation (e.g. another writer changed it) stops the rollout.

## Rollback snapshot (OPS-03) and removals (OPS-02)

-   Before wave 1, save the current value of **every field the job touches** for
    every target entity: status, budget amount, target, bid strategy,
    listing-group tree, negative lists. Save it outside Google Ads (warehouse
    table keyed by job ID).
-   Build the **restore payload** from the snapshot and dry-run it with
    `validate_only`. A snapshot that can't be restored isn't a rollback plan.
-   **Removals are one-way**: removed campaigns and other removed entities can't
    be re-enabled. Prefer pause. If removal is required, snapshot the full
    configuration (so it can be recreated) and get ≥ T2 approval for that part
    (OPS-02); in a bulk job the job is T3 anyway.
-   `change_event` (30 days) gives old and new values for many resource types,
    but not for asset groups, shared sets or listing-group filters; don't rely
    on it as the only snapshot.

## Labels and identity (STR-04, OPS-06)

-   Create one label per job (e.g. `agent` plus a job ID label, per
    `client_rules.md`) and attach it to every campaign, ad group, ad or
    criterion the job creates or changes.
-   In API v24 labels attach to customers, campaigns, ad groups, ads and ad
    group criteria. **Asset groups have no labels**: use the naming convention
    (`_AGENT_` + job ID) and the campaign label.
-   Run the job under the agent's dedicated login identity (OPS-06) so
    `change_event.user_email` and `client_type = GOOGLE_ADS_API` separate it
    from people and from other tools.

## Worked incident: 20M operations, 3% failed

-   **Findings:** EXE-03 BLOCK (3% > 2%: should have halted after the first
    wave); EXE-04 BLOCK if it went out without waves; EXE-02 BLOCK unless the
    ~770,000 failed operations are reconciled; EXE-06 if scheduled reports lost
    quota at the same time.
-   **Clean-up plan (options):**
    1.  Freeze further writes. Build the ledger from the batch results; group
        failures by error code; fix the top codes in the generator.
    2.  Read back a sample of "successful" entities to confirm they match.
    3.  Re-validate the fixed operations and re-run them as a small wave with
        the gate.
    4.  If the job's purpose fails the guardrails (e.g. it created many small
        campaigns that fail STR-01), propose rolling back from the snapshot
        instead of repairing.
