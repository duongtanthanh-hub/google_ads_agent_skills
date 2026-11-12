# 1. Write method and API limits

## Limits to plan around

Limit                                          | Value                                                                                                                                                                                                                          | Src
---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ---
Operations per synchronous mutate request      | 10,000 (over it → `RequestError.TOO_MANY_MUTATE_OPERATIONS`)                                                                                                                                                                   | P (Google Ads API: system limits / mutate best practices)
Operations per `AddBatchJobOperations` request | 10,000                                                                                                                                                                                                                         | P (Google Ads Developer Blog, batch processing; confirm in the current guide)
Operations per batch job                       | 1,000,000                                                                                                                                                                                                                      | P (same; confirm)
Request size                                   | `AddBatchJobOperations` requests over about 10 MB (10,484,504 bytes in the v24 error description) fail with `BatchJobError.REQUEST_TOO_LARGE`; split by size as well as by count (image-heavy asset operations hit this first) | P (API reference, v24)
Batch job running time                         | Capped by `batch_job.metadata.execution_limit_seconds`; a job running longer is cancelled                                                                                                                                      | P (API reference)
Daily operations                               | By access level of the Google Cloud project: Explorer 2,880/day on production accounts; Basic 15,000/day; Standard unlimited (rate limits still apply)                                                                         | P (Access levels and permissible use)

Check limits against the current API version before each new job type; they
change between versions.

## Choosing the method

Use                          | When                                                                                       | Notes
---------------------------- | ------------------------------------------------------------------------------------------ | -----
Chunked synchronous `Mutate` | Up to tens of thousands of simple operations; you need results at once (e.g. a pilot wave) | ≤ 10,000 operations per request (EXE-02); set `partial_failure = true` unless the chunk must be all-or-nothing; one writer per campaign/budget
`BatchJobService`            | Large jobs, many entity types, or create-with-children using temporary IDs                 | Always partial failure; asynchronous; results via `ListBatchJobResults`
Neither                      | The job creates many small campaigns that would each fail STR-01                           | Fix the plan (consolidate), not the method

## How `BatchJobService` works (for planning)

1.  `MutateBatchJob` creates the job (status `PENDING`).
2.  `AddBatchJobOperations` uploads operations in chunks of ≤ 10,000, passing
    the `sequence_token` from the previous response (`next_sequence_token`). A
    wrong token fails with `INVALID_SEQUENCE_TOKEN`.
3.  `RunBatchJob` starts it (long-running operation). After that, the job can't
    be changed (`CANNOT_MODIFY_JOB_AFTER_JOB_STARTS_RUNNING`).
4.  Poll with exponential backoff, not in a tight loop. Read `batch_job.status`
    (`PENDING`, `RUNNING`, `DONE`), `metadata.estimated_completion_ratio`,
    `metadata.operation_count` and `metadata.executed_operation_count`.
5.  `ListBatchJobResults` once `DONE` (earlier → `RESULTS_NOT_READY`). Each
    result has `operation_index`, `mutate_operation_response` and `status` (a
    non-OK `status` = that operation failed).
6.  If `executed_operation_count` < `operation_count`, the job didn't finish
    (e.g. it hit the execution limit). Treat the rest as not applied.

**Ordering for speed and fewer errors** (P, batch processing best practices):
group operations by type (campaigns, then ad groups/asset groups, then
criteria/listing groups) and, within a type, by parent.

**Atomic groups inside a batch job** (they succeed or fail together):

-   consecutive `AssetGroupListingGroupFilterOperation`s for the same asset
    group (`ASSET_GROUP_LISTING_GROUP_FILTER_TRANSACTION_FAILURE`);
-   an asset group create followed by its asset links
    (`ASSET_GROUP_AND_ASSET_GROUP_ASSET_TRANSACTION_FAILURE`);
-   a campaign create with its campaign assets
    (`CAMPAIGN_AND_CAMPAIGN_ASSET_TRANSACTION_FAILURE`).

These three are the `BatchJobError` transaction types in v24. Standard Shopping
listing-group trees aren't a documented atomic group, but a tree must be valid
as a whole: keep each ad group's tree in one chunk and check the whole tree
after the wave.

One bad operation fails its whole group; count failures by group as well as by
operation when you compute EXE-03.

## Synchronous mutate details

-   `validate_only = true`: validated, not executed; mutates return errors only
    (no results). Use it for the dry run (EXE-01).
-   `partial_failure = true`: valid operations run; errors come back in
    `partial_failure_error` with the operation index. With `false`, the request
    is all-or-nothing.
-   `response_content_type = MUTABLE_RESOURCE` returns the changed resource, a
    cheap read-back for small waves.
-   Temporary (negative) IDs let one request create a parent and its children.

## Concurrency

-   Writes to the same entity or parent in parallel fail with
    `DatabaseError.CONCURRENT_MODIFICATION`. Shard work by campaign (or budget),
    one worker per shard.
-   Keep the number of parallel writers low and fixed; raise it only if the
    error rate and `RESOURCE_TEMPORARILY_EXHAUSTED` rate stay low (see
    [ads-api-hygiene](../../ads-api-hygiene/SKILL.md)).
