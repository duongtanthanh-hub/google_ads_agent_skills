# Read-only GAQL queries (Google Ads API v24 field names)

Run through the MCP `search` tool or `GoogleAdsService.SearchStream`. `{IDS}` =
the changed campaigns; `{CONTROL_IDS}` = untouched comparable campaigns. Dates
in the account's time zone.

## Q0. Account time zone and currency

```sql
SELECT customer.id, customer.descriptive_name, customer.time_zone,
       customer.currency_code
FROM customer
```

## Q1. Read-back and serving (campaigns)

```sql
SELECT campaign.id, campaign.name, campaign.status, campaign.primary_status,
       campaign.primary_status_reasons, campaign.bidding_strategy_type,
       campaign.bidding_strategy_system_status,
       campaign.maximize_conversion_value.target_roas,
       campaign.maximize_conversions.target_cpa_micros,
       campaign.target_roas.target_roas,
       campaign_budget.amount_micros
FROM campaign
WHERE campaign.id IN ({IDS})
```

## Q2. Asset groups, assets and ads: status and policy

```sql
SELECT campaign.id, asset_group.id, asset_group.status,
       asset_group.primary_status, asset_group.primary_status_reasons
FROM asset_group
WHERE campaign.id IN ({IDS})
```

```sql
SELECT asset_group.id, asset_group_asset.field_type, asset_group_asset.status,
       asset_group_asset.policy_summary.approval_status,
       asset_group_asset.policy_summary.review_status
FROM asset_group_asset
WHERE campaign.id IN ({IDS})
  AND asset_group_asset.status != 'REMOVED'
```

```sql
SELECT campaign.id, ad_group.id, ad_group_ad.status,
       ad_group_ad.policy_summary.approval_status,
       ad_group_ad.policy_summary.review_status
FROM ad_group_ad
WHERE campaign.id IN ({IDS})
  AND ad_group_ad.status != 'REMOVED'
```

## Q3. Daily delivery and results for the changed scope

```sql
SELECT segments.date, campaign.id, metrics.impressions, metrics.clicks,
       metrics.cost_micros, metrics.conversions, metrics.conversions_value,
       metrics.search_budget_lost_impression_share
FROM campaign
WHERE campaign.id IN ({IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

Run for the post window and, if there is no D0 snapshot, for the pre window.
Save results with the date you read them (maturity).

## Q4. Same for control campaigns

Q3 with `campaign.id IN ({CONTROL_IDS})`.

## Q5a. Conversion action mix (no cost metrics with this segment)

```sql
SELECT segments.conversion_action, segments.conversion_action_category,
       campaign.id, metrics.conversions, metrics.conversions_value
FROM campaign
WHERE campaign.id IN ({IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

## Q5b. Conversion lag (OPS-10)

```sql
SELECT segments.conversion_lag_bucket, metrics.conversions,
       metrics.conversions_value
FROM campaign
WHERE campaign.id IN ({IDS})
  AND segments.date BETWEEN '{TODAY_MINUS_120}' AND '{TODAY_MINUS_30}'
```

The window ends 30 days ago so the clicks in it have had time to convert; newer
days would understate late conversions.

## Q5c. Tracking check: conversions by conversion date vs clicks

```sql
SELECT segments.date, metrics.clicks, metrics.conversions_by_conversion_date,
       metrics.conversions_value_by_conversion_date
FROM campaign
WHERE campaign.id IN ({IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

## Q6. Confounders: other changes to the scope since D0

```sql
SELECT change_event.change_date_time, change_event.change_resource_type,
       change_event.resource_change_operation, change_event.changed_fields,
       change_event.client_type, change_event.campaign
FROM change_event
WHERE change_event.change_date_time >= '{D0}'
  AND change_event.change_date_time <= '{NOW}'
  AND change_event.campaign IN ('customers/{CID}/campaigns/{CAMPAIGN_ID}')
LIMIT 10000
```

`change_event` keeps 30 days and doesn't cover asset groups, shared sets or
listing-group filters. Add a `change_status` query
(`resource_type IN ('ASSET_GROUP', 'SHARED_SET', 'CAMPAIGN_SHARED_SET')`, same
window, `LIMIT 10000`) for asset groups and negative lists; listing-group filter
changes are in neither, so check the executing agent's change log.
