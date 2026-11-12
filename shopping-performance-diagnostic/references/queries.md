# Read-only queries (Google Ads API v24 field names)

All queries are reads. Placeholders: `{CID}`, `{CAMPAIGN_IDS}`, `{CAMPAIGN_ID}`,
dates as `'YYYY-MM-DD'`. Run period queries once per window (DX-01). Check new
fields, and which resources a metric can be selected with, using
`get_resource_metadata`. Use `SearchStream` for large results.
`metrics.cost_micros / 1,000,000` = cost in account currency. Numbers follow the
PMax diagnostic where the query is the same; Q4 and Q9 aren't used here.

## Q0. Accounts (pre-flight)

```sql
SELECT customer_client.id, customer_client.descriptive_name,
       customer_client.status, customer_client.manager,
       customer_client.currency_code, customer_client.time_zone
FROM customer_client
WHERE customer_client.status = 'ENABLED' AND customer_client.manager = FALSE
```

## Q1. Campaign status, bidding, caps, priority, budget

```sql
SELECT campaign.id, campaign.name, campaign.status, campaign.primary_status,
       campaign.primary_status_reasons, campaign.bidding_strategy_type,
       campaign.bidding_strategy_system_status, campaign.bidding_strategy,
       campaign.target_spend.cpc_bid_ceiling_micros,
       campaign.target_roas.target_roas,
       campaign.target_roas.cpc_bid_ceiling_micros,
       campaign.maximize_conversion_value.target_roas,
       campaign.campaign_budget, campaign_budget.amount_micros,
       campaign.shopping_setting.merchant_id,
       campaign.shopping_setting.feed_label,
       campaign.shopping_setting.campaign_priority, campaign.start_date_time
FROM campaign
WHERE campaign.advertising_channel_type = 'SHOPPING'
  AND campaign.status != 'REMOVED'
```

## Q1b. Ad groups

```sql
SELECT campaign.id, ad_group.id, ad_group.name, ad_group.status,
       ad_group.type, ad_group.primary_status, ad_group.cpc_bid_micros
FROM ad_group
WHERE campaign.advertising_channel_type = 'SHOPPING'
  AND ad_group.status != 'REMOVED'
```

## Q1c. Portfolio strategies (when `campaign.bidding_strategy` is set)

```sql
SELECT bidding_strategy.id, bidding_strategy.name, bidding_strategy.type,
       bidding_strategy.target_spend.cpc_bid_ceiling_micros,
       bidding_strategy.target_roas.target_roas,
       bidding_strategy.maximize_conversion_value.target_roas
FROM bidding_strategy
```

## Q2. Daily trend

```sql
SELECT campaign.id, campaign.name, campaign.advertising_channel_type,
       segments.date, metrics.impressions, metrics.clicks,
       metrics.cost_micros, metrics.conversions, metrics.conversions_value
FROM campaign
WHERE campaign.advertising_channel_type IN ('SHOPPING', 'PERFORMANCE_MAX')
  AND segments.date BETWEEN '{BASE_START}' AND '{CUR_END}'
```

## Q3. Conversion mix by action (no cost in this query)

```sql
SELECT campaign.id, segments.conversion_action_name,
       segments.conversion_action_category, metrics.conversions,
       metrics.conversions_value
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

## Q3b. Goals and conversion-action settings

```sql
SELECT campaign.id, campaign_conversion_goal.category,
       campaign_conversion_goal.origin, campaign_conversion_goal.biddable
FROM campaign_conversion_goal
WHERE campaign.id IN ({CAMPAIGN_IDS})
```

```sql
SELECT conversion_action.id, conversion_action.name, conversion_action.category,
       conversion_action.primary_for_goal,
       conversion_action.include_in_conversions_metric,
       conversion_action.value_settings.always_use_default_value
FROM conversion_action
WHERE conversion_action.status = 'ENABLED'
```

## Q3c. Product-group tree, bids and exclusions

```sql
SELECT campaign.id, ad_group.id, ad_group_criterion.criterion_id,
       ad_group_criterion.status, ad_group_criterion.negative,
       ad_group_criterion.listing_group.type,
       ad_group_criterion.listing_group.case_value.product_type.value,
       ad_group_criterion.listing_group.case_value.product_custom_attribute.index,
       ad_group_criterion.listing_group.case_value.product_custom_attribute.value,
       ad_group_criterion.listing_group.case_value.product_item_id.value,
       ad_group_criterion.cpc_bid_micros,
       ad_group_criterion.effective_cpc_bid_micros
FROM ad_group_criterion
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND ad_group_criterion.type = 'LISTING_GROUP'
  AND ad_group_criterion.status != 'REMOVED'
```

## Q3d. Benchmark signals (check the resource with `get_resource_metadata`)

```sql
SELECT campaign.id, ad_group.id, ad_group_criterion.criterion_id,
       metrics.ctr, metrics.average_cpc, metrics.benchmark_ctr,
       metrics.benchmark_average_max_cpc
FROM product_group_view
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

If the API rejects the benchmark metrics here, mark them NOT_ASSESSED.

## Q5. Product-group performance and impression share

```sql
SELECT campaign.id, ad_group.id, ad_group_criterion.criterion_id,
       metrics.impressions, metrics.clicks, metrics.cost_micros,
       metrics.conversions, metrics.conversions_value,
       metrics.search_impression_share,
       metrics.search_rank_lost_impression_share
FROM product_group_view
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

Join to Q3c on `criterion_id` for the dimension, bid and `negative` flag.

## Q6. Product performance (Shopping and PMax, for overlap)

```sql
SELECT campaign.id, campaign.advertising_channel_type,
       segments.product_item_id, segments.product_title,
       segments.product_type_l1, segments.product_custom_attribute0,
       segments.product_feed_label, metrics.impressions, metrics.clicks,
       metrics.cost_micros, metrics.conversions, metrics.conversions_value
FROM shopping_performance_view
WHERE campaign.advertising_channel_type IN ('SHOPPING', 'PERFORMANCE_MAX')
  AND segments.date BETWEEN '{START}' AND '{END}'
ORDER BY metrics.cost_micros DESC
LIMIT 5000
```

## Q7. Product eligibility (one campaign)

```sql
SELECT shopping_product.item_id, shopping_product.title,
       shopping_product.status, shopping_product.issues,
       shopping_product.feed_label
FROM shopping_product
WHERE shopping_product.campaign = 'customers/{CID}/campaigns/{CAMPAIGN_ID}'
```

Needs the equality filter on the campaign; `issues` can lag up to 24 hours.

## Q8. Change history (last 30 days max)

```sql
SELECT change_event.change_date_time, change_event.change_resource_type,
       change_event.change_resource_name,
       change_event.resource_change_operation, change_event.changed_fields,
       change_event.client_type, change_event.campaign,
       change_event.old_resource, change_event.new_resource
FROM change_event
WHERE change_event.change_date_time >= '{START}'
  AND change_event.change_date_time <= '{END}'
  AND change_event.change_resource_type IN ('CAMPAIGN', 'CAMPAIGN_BUDGET',
      'AD_GROUP', 'AD_GROUP_CRITERION')
LIMIT 10000
```

No `metrics.*`; dates within the last 30 days; `LIMIT` ≤ 10,000. Match budget
rows to campaigns via `campaign.campaign_budget` (Q1).

## Q10. Impression share (campaign; repeat on `ad_group` for ad-group level)

```sql
SELECT campaign.id, metrics.search_impression_share,
       metrics.search_budget_lost_impression_share,
       metrics.search_rank_lost_impression_share,
       metrics.search_absolute_top_impression_share,
       metrics.search_click_share
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

0.0999 means "< 10%"; 0.9001 means "> 90%"; missing = NOT_ASSESSED.

## Q11. Conversion lag (last 90 days)

```sql
SELECT campaign.id, segments.conversion_lag_bucket, metrics.conversions,
       metrics.conversions_value
FROM campaign
WHERE campaign.advertising_channel_type = 'SHOPPING'
  AND segments.date BETWEEN '{START_90D}' AND '{END}'
```

## Q11b. Click date vs conversion date

```sql
SELECT campaign.id, segments.date, metrics.conversions_value,
       metrics.conversions_value_by_conversion_date
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date BETWEEN '{START}' AND '{END}'
```

## Q12. Seasonality adjustments and data exclusions

```sql
SELECT bidding_seasonality_adjustment.name,
       bidding_seasonality_adjustment.scope,
       bidding_seasonality_adjustment.start_date_time,
       bidding_seasonality_adjustment.end_date_time,
       bidding_seasonality_adjustment.conversion_rate_modifier,
       bidding_seasonality_adjustment.campaigns,
       bidding_seasonality_adjustment.advertising_channel_types
FROM bidding_seasonality_adjustment
WHERE bidding_seasonality_adjustment.status = 'ENABLED'
```

```sql
SELECT bidding_data_exclusion.name, bidding_data_exclusion.scope,
       bidding_data_exclusion.start_date_time,
       bidding_data_exclusion.end_date_time, bidding_data_exclusion.campaigns,
       bidding_data_exclusion.advertising_channel_types
FROM bidding_data_exclusion
WHERE bidding_data_exclusion.status = 'ENABLED'
```

## Q13. Shopping recommendations (context only; never apply from this skill)

```sql
SELECT recommendation.type, recommendation.campaign,
       recommendation.dismissed
FROM recommendation
WHERE recommendation.type IN ('SHOPPING_FIX_DISAPPROVED_PRODUCTS',
      'SHOPPING_ADD_PRODUCTS_TO_CAMPAIGN',
      'SHOPPING_FIX_SUSPENDED_MERCHANT_CENTER_ACCOUNT',
      'SHOPPING_FIX_MERCHANT_CENTER_ACCOUNT_SUSPENSION_WARNING',
      'CAMPAIGN_BUDGET')
```

## M1. Price benchmarks (Merchant API reports, not Google Ads API)

Only with existing Merchant API read access. Merchant Center Query Language,
`accounts/{MERCHANT_ID}/reports:search`; check field names against the Merchant
API reports reference.

```sql
SELECT report_country_code, id, offer_id, title, price, benchmark_price
FROM price_competitiveness_product_view
WHERE report_country_code = '{COUNTRY}'
```
