# Read-only GAQL queries (Google Ads API v24 field names)

All queries are reads, run through the MCP `search` tool (arguments
`customer_id`, `query`) or `GoogleAdsService.SearchStream`. Replace
`{CAMPAIGN_IDS}` with a comma-separated list. Check new field names with
`get_resource_metadata` before use. `metrics.cost_micros` / 1,000,000 = account
currency.

## Q0. Account selection (when no customer ID is given)

```sql
SELECT customer_client.id, customer_client.descriptive_name,
       customer_client.status, customer_client.manager
FROM customer_client
WHERE customer_client.status = 'ENABLED' AND customer_client.manager = FALSE
```

Run under each ID from `list_accessible_customers`. Don't query manager or
disabled accounts.

## Q1. Current state of the campaigns in scope

```sql
SELECT campaign.id, campaign.name, campaign.status, campaign.primary_status,
       campaign.primary_status_reasons, campaign.advertising_channel_type,
       campaign.bidding_strategy_type, campaign.bidding_strategy_system_status,
       campaign.bidding_strategy,
       campaign.maximize_conversion_value.target_roas,
       campaign.maximize_conversions.target_cpa_micros,
       campaign.target_roas.target_roas,
       campaign.target_spend.cpc_bid_ceiling_micros,
       campaign.start_date_time, campaign.labels,
       campaign_budget.id, campaign_budget.amount_micros,
       campaign_budget.explicitly_shared, campaign_budget.reference_count
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
```

`campaign.bidding_strategy` is set only for portfolio strategies (then use Q4).

## Q2. Last changes of the same kind (CHG-01…03), last 30 days

```sql
SELECT change_event.change_date_time, change_event.change_resource_type,
       change_event.change_resource_name, change_event.resource_change_operation,
       change_event.changed_fields, change_event.client_type,
       change_event.old_resource, change_event.new_resource,
       change_event.campaign
FROM change_event
WHERE change_event.change_date_time >= '{START_DATE}'
  AND change_event.change_date_time <= '{END_DATE}'
  AND change_event.change_resource_type IN ('CAMPAIGN', 'CAMPAIGN_BUDGET')
ORDER BY change_event.change_date_time DESC
LIMIT 10000
```

-   Filter the rows to the campaign (`change_event.campaign`) or, for budgets,
    to the budget's resource name (`change_event.change_resource_name =
    'customers/{CID}/campaignBudgets/{BUDGET_ID}'`).
-   Read `changed_fields` to tell a budget amount, target, status or strategy
    change apart.
-   The date range must be within the last 30 days and `LIMIT` ≤ 10,000. No
    matching row → "≥ 30 days since the last change".
-   Don't select `change_event.user_email` unless the task needs it.

## Q2b. Asset group and shared-list changes (not in change_event)

```sql
SELECT change_status.resource_name, change_status.last_change_date_time,
       change_status.resource_type, change_status.resource_status,
       change_status.campaign, change_status.asset_group
FROM change_status
WHERE change_status.last_change_date_time >= '{START_DATE}'
  AND change_status.last_change_date_time <= '{END_DATE}'
  AND change_status.resource_type = 'ASSET_GROUP'
LIMIT 10000
```

`change_status` needs a date filter within the last 90 days and `LIMIT` ≤
10,000. It shows the latest change only, without old/new values. For shared
negative lists, use `resource_type IN ('SHARED_SET', 'CAMPAIGN_SHARED_SET')` and
select `change_status.shared_set`. Listing-group filters are in neither
`change_status` nor `change_event`; use the executing agent's change log.

## Q3. 30-day volume and efficiency (BUD-01, BID-02, MEA-06)

```sql
SELECT campaign.id, campaign.name, metrics.cost_micros, metrics.conversions,
       metrics.conversions_value, metrics.search_budget_lost_impression_share
FROM campaign
WHERE segments.date BETWEEN '{START}' AND '{END}'
  AND campaign.id IN ({CAMPAIGN_IDS})
```

Set `{END}` before the conversion-lag window (CHG-04, lag per OPS-10) and
`{START}` 30 days earlier. For BID-02 run it over comparable non-brand campaigns
instead of `{CAMPAIGN_IDS}`.

## Q4. Portfolio strategies (blast radius of a target change)

```sql
SELECT bidding_strategy.id, bidding_strategy.name, bidding_strategy.type,
       bidding_strategy.status, bidding_strategy.campaign_count,
       bidding_strategy.maximize_conversion_value.target_roas,
       bidding_strategy.target_roas.target_roas
FROM bidding_strategy
WHERE bidding_strategy.id = {STRATEGY_ID}
```

## Q5. Shared negative lists and where they apply

```sql
SELECT shared_set.id, shared_set.name, shared_set.type, shared_set.status,
       shared_set.member_count, shared_set.reference_count
FROM shared_set
WHERE shared_set.type IN ('NEGATIVE_KEYWORDS', 'ACCOUNT_LEVEL_NEGATIVE_KEYWORDS')
```

```sql
SELECT shared_criterion.shared_set, shared_criterion.keyword.text,
       shared_criterion.keyword.match_type
FROM shared_criterion
WHERE shared_criterion.shared_set = 'customers/{CID}/sharedSets/{SHARED_SET_ID}'
```

## Q6. New campaigns in the last 7 days (STR-02)

```sql
SELECT change_event.change_date_time, change_event.change_resource_name,
       change_event.client_type
FROM change_event
WHERE change_event.change_date_time >= '{SEVEN_DAYS_AGO}'
  AND change_event.change_date_time <= '{NOW}'
  AND change_event.change_resource_type = 'CAMPAIGN'
  AND change_event.resource_change_operation = 'CREATE'
LIMIT 10000
```

## Q7. Converting search terms (check candidate negatives)

```sql
SELECT campaign_search_term_view.search_term, campaign.id,
       metrics.clicks, metrics.conversions, metrics.conversions_value
FROM campaign_search_term_view
WHERE segments.date DURING LAST_30_DAYS
  AND metrics.conversions > 0
```

Check which campaign types this view returns in your API version (it is meant to
cover more types than `search_term_view`). Match each candidate negative (with
its match type) against these terms.
