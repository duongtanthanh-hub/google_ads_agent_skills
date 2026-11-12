# Read-only GAQL queries (Google Ads API v24 field names)

All queries are reads (MCP `search` tool or `GoogleAdsService.SearchStream`).
Each request counts as one operation toward the daily quota; include them in the
forecast. Check new fields with `get_resource_metadata` first.

## Q0. Accounts in scope

```sql
SELECT customer_client.id, customer_client.descriptive_name,
       customer_client.status, customer_client.manager
FROM customer_client
WHERE customer_client.status = 'ENABLED' AND customer_client.manager = FALSE
```

## Q1. Sizing: active campaigns and recent launches (STR-02)

```sql
SELECT campaign.id, campaign.name, campaign.status,
       campaign.advertising_channel_type, campaign.start_date_time
FROM campaign
WHERE campaign.status = 'ENABLED'
```

```sql
SELECT change_event.change_date_time, change_event.change_resource_name
FROM change_event
WHERE change_event.change_date_time >= '{SEVEN_DAYS_AGO}'
  AND change_event.change_date_time <= '{NOW}'
  AND change_event.change_resource_type = 'CAMPAIGN'
  AND change_event.resource_change_operation = 'CREATE'
LIMIT 10000
```

## Q2. Batch job progress and size

```sql
SELECT batch_job.id, batch_job.status,
       batch_job.metadata.creation_date_time,
       batch_job.metadata.start_date_time,
       batch_job.metadata.completion_date_time,
       batch_job.metadata.estimated_completion_ratio,
       batch_job.metadata.operation_count,
       batch_job.metadata.executed_operation_count,
       batch_job.metadata.execution_limit_seconds
FROM batch_job
WHERE batch_job.id = {BATCH_JOB_ID}
```

Per-operation results come from `BatchJobService.ListBatchJobResults`
(`operation_index`, `status`), not from GAQL. Poll with backoff.

## Q3. Read-back of a wave by label (OPS-05)

```sql
SELECT label.id, label.name, label.status
FROM label
WHERE label.name = '{JOB_LABEL}'
```

```sql
SELECT campaign.id, campaign.name, campaign.status, campaign.primary_status,
       campaign.bidding_strategy_type,
       campaign.maximize_conversion_value.target_roas,
       campaign_budget.amount_micros, campaign.labels
FROM campaign
WHERE campaign.labels CONTAINS ANY ('customers/{CID}/labels/{LABEL_ID}')
```

For ad groups, use `ad_group.labels` the same way. Asset groups have no labels:
filter by name (`asset_group.name LIKE '%{JOB_ID}%'`) within the labelled
campaigns.

## Q4. What actually changed since the wave started (all resource types)

```sql
SELECT change_status.resource_name, change_status.last_change_date_time,
       change_status.resource_type, change_status.resource_status,
       change_status.campaign, change_status.ad_group,
       change_status.asset_group
FROM change_status
WHERE change_status.last_change_date_time >= '{WAVE_START}'
  AND change_status.last_change_date_time <= '{NOW}'
LIMIT 10000
```

Within 90 days; `LIMIT` ≤ 10,000. If 10,000 rows come back, narrow the time
window and repeat. Changes can take a few minutes to appear. Listing-group
filters aren't a `change_status` resource type; check them through the job's own
results and a read-back of the tree.

## Q5. Who made the changes (identity check, OPS-06)

```sql
SELECT change_event.change_date_time, change_event.change_resource_type,
       change_event.resource_change_operation, change_event.client_type,
       change_event.user_email, change_event.campaign
FROM change_event
WHERE change_event.change_date_time >= '{WAVE_START}'
  AND change_event.change_date_time <= '{NOW}'
LIMIT 10000
```

Select `user_email` only for this identity check; don't copy it into reports.

## Q6. D+1 health of wave entities (OPS-08)

```sql
SELECT campaign.id, asset_group.id, asset_group.name, asset_group.status,
       asset_group.primary_status, asset_group.primary_status_reasons
FROM asset_group
WHERE campaign.labels CONTAINS ANY ('customers/{CID}/labels/{LABEL_ID}')
```

```sql
SELECT campaign.id, ad_group.id, ad_group_ad.status,
       ad_group_ad.policy_summary.approval_status,
       ad_group_ad.policy_summary.review_status
FROM ad_group_ad
WHERE campaign.labels CONTAINS ANY ('customers/{CID}/labels/{LABEL_ID}')
  AND ad_group_ad.status != 'REMOVED'
```

For full D+1 checks use
[ads-post-change-check](../../ads-post-change-check/SKILL.md). If a
`campaign.labels` filter is rejected from these resources in your API version,
filter by the campaign IDs returned by Q3 instead.
