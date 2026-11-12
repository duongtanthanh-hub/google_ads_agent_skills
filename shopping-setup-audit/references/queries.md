# Read-only GAQL queries (Google Ads API v24 field names)

All queries are reads. Replace `{CID}`, `{CAMPAIGN_IDS}` (comma-separated),
`{CAMPAIGN_ID}`, `{START_DATE}`/`{END_DATE}` (`YYYY-MM-DD`) and `{FEED_LABEL}`.
Check field names with `get_resource_metadata` or your client library, and use
`SearchStream` for large results. Money fields are micros (÷ 1,000,000).

## Q0. Pre-flight: enabled client accounts

```sql
SELECT customer_client.id, customer_client.descriptive_name,
       customer_client.status, customer_client.manager
FROM customer_client
WHERE customer_client.status = 'ENABLED' AND customer_client.manager = FALSE
```

## Q0b. Linked Merchant Center accounts

```sql
SELECT product_link.product_link_id, product_link.type,
       product_link.merchant_center.merchant_center_id
FROM product_link
WHERE product_link.type = 'MERCHANT_CENTER'
```

## Q1. Shopping campaign settings

```sql
SELECT campaign.id, campaign.name, campaign.status, campaign.primary_status,
       campaign.primary_status_reasons, campaign.advertising_channel_sub_type,
       campaign.bidding_strategy_type, campaign.bidding_strategy,
       campaign.bidding_strategy_system_status,
       campaign.target_spend.cpc_bid_ceiling_micros,
       campaign.maximize_conversion_value.target_roas,
       campaign.maximize_conversion_value.cpc_bid_ceiling_micros,
       campaign.target_roas.target_roas,
       campaign.target_roas.cpc_bid_ceiling_micros,
       campaign.shopping_setting.merchant_id,
       campaign.shopping_setting.feed_label,
       campaign.shopping_setting.campaign_priority,
       campaign.shopping_setting.enable_local,
       campaign.shopping_setting.ignore_brand_exclusion_in_shopping_ads,
       campaign_budget.amount_micros, campaign_budget.explicitly_shared,
       campaign.labels, campaign.start_date_time
FROM campaign
WHERE campaign.advertising_channel_type = 'SHOPPING'
  AND campaign.id IN ({CAMPAIGN_IDS})
```

## Q1b. Portfolio bid strategies (if `campaign.bidding_strategy` is set)

```sql
SELECT bidding_strategy.id, bidding_strategy.name, bidding_strategy.type,
       bidding_strategy.target_spend.cpc_bid_ceiling_micros,
       bidding_strategy.target_roas.target_roas,
       bidding_strategy.target_roas.cpc_bid_ceiling_micros,
       bidding_strategy.maximize_conversion_value.target_roas,
       bidding_strategy.maximize_conversion_value.cpc_bid_ceiling_micros
FROM bidding_strategy
```

## Q2. Baseline: comparable Shopping and PMax campaigns

Use a window that ends before the conversion-lag window (CHG-04), e.g. the 30
days ending 7 days ago.

```sql
SELECT campaign.id, campaign.name, campaign.advertising_channel_type,
       campaign.bidding_strategy_type, campaign.shopping_setting.feed_label,
       metrics.cost_micros, metrics.clicks, metrics.average_cpc,
       metrics.conversions, metrics.conversions_value
FROM campaign
WHERE campaign.advertising_channel_type IN ('SHOPPING', 'PERFORMANCE_MAX')
  AND segments.date BETWEEN '{START_DATE}' AND '{END_DATE}'
  AND metrics.cost_micros > 0
```

## Q3. Conversion goals and actions

```sql
SELECT campaign.id, campaign_conversion_goal.category,
       campaign_conversion_goal.origin, campaign_conversion_goal.biddable
FROM campaign_conversion_goal
WHERE campaign.id IN ({CAMPAIGN_IDS})
```

```sql
SELECT conversion_action.id, conversion_action.name, conversion_action.status,
       conversion_action.category, conversion_action.primary_for_goal,
       conversion_action.value_settings.default_value,
       conversion_action.value_settings.always_use_default_value,
       conversion_action.attribution_model_settings.attribution_model
FROM conversion_action
WHERE conversion_action.status = 'ENABLED'
```

## Q3b. What the campaign counts as a conversion (SHP-06)

No `metrics.cost_micros` here: it can't be combined with conversion-action
segments.

```sql
SELECT campaign.id, segments.conversion_action_category,
       segments.conversion_action_name, metrics.conversions,
       metrics.conversions_value, metrics.all_conversions
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date BETWEEN '{START_DATE}' AND '{END_DATE}'
```

## Q4. Ad groups and product-group trees

```sql
SELECT campaign.id, ad_group.id, ad_group.name, ad_group.status, ad_group.type,
       ad_group_criterion.criterion_id, ad_group_criterion.status,
       ad_group_criterion.negative, ad_group_criterion.listing_group.type,
       ad_group_criterion.listing_group.parent_ad_group_criterion,
       ad_group_criterion.listing_group.case_value.product_custom_attribute.index,
       ad_group_criterion.listing_group.case_value.product_custom_attribute.value,
       ad_group_criterion.listing_group.case_value.product_type.level,
       ad_group_criterion.listing_group.case_value.product_type.value,
       ad_group_criterion.listing_group.case_value.product_brand.value,
       ad_group_criterion.listing_group.case_value.product_item_id.value,
       ad_group_criterion.cpc_bid_micros,
       ad_group_criterion.effective_cpc_bid_micros
FROM ad_group_criterion
WHERE ad_group_criterion.type = 'LISTING_GROUP'
  AND campaign.id IN ({CAMPAIGN_IDS})
  AND ad_group_criterion.status != 'REMOVED'
```

A unit node with empty `case_value` fields is the "Everything else" node of its
parent.

## Q5. Inventory filter (campaign listing scope)

```sql
SELECT campaign.id, campaign_criterion.criterion_id, campaign_criterion.type,
       campaign_criterion.listing_scope.dimensions
FROM campaign_criterion
WHERE campaign_criterion.type = 'LISTING_SCOPE'
  AND campaign.id IN ({CAMPAIGN_IDS})
```

## Q6. Products included by one campaign (status and issues)

```sql
SELECT shopping_product.item_id, shopping_product.title,
       shopping_product.feed_label, shopping_product.language_code,
       shopping_product.status, shopping_product.issues,
       shopping_product.custom_attribute0, shopping_product.product_type_level1,
       shopping_product.effective_max_cpc_micros
FROM shopping_product
WHERE shopping_product.campaign = 'customers/{CID}/campaigns/{CAMPAIGN_ID}'
```

Campaign scope **requires the equality filter** on `shopping_product.campaign`
(one campaign per query); without it you get every product in the linked
Merchant Center accounts. `status` and `issues` can take up to 24 hours to
update, and `issues` can't be filtered on, so group by `issues.error_code` after
reading. `effective_max_cpc_micros` is filled only for manual bidding.

## Q6b. Overlap candidates: retail campaigns on the same feed

```sql
SELECT campaign.id, campaign.name, campaign.status,
       campaign.advertising_channel_type,
       campaign.shopping_setting.merchant_id,
       campaign.shopping_setting.feed_label,
       campaign.shopping_setting.campaign_priority
FROM campaign
WHERE campaign.advertising_channel_type IN ('SHOPPING', 'PERFORMANCE_MAX')
  AND campaign.status = 'ENABLED'
```

Run Q6 for each candidate on the same Merchant Center ID and feed label, and
intersect the `item_id` sets. Also compare location targeting: overlap only
matters for the same country.

## Q6c. Account-scope products for one feed label (FEED-04 comparison)

```sql
SELECT shopping_product.item_id, shopping_product.status
FROM shopping_product
WHERE shopping_product.feed_label = '{FEED_LABEL}'
```

## Q7. Product performance by attribute (STR-01 projections, SHP-02 CPCs)

```sql
SELECT campaign.id, segments.product_type_l1,
       segments.product_custom_attribute0, segments.product_feed_label,
       metrics.clicks, metrics.cost_micros, metrics.average_cpc,
       metrics.conversions, metrics.conversions_value
FROM shopping_performance_view
WHERE segments.date BETWEEN '{START_DATE}' AND '{END_DATE}'
  AND metrics.clicks > 0
```

Swap `segments.product_custom_attribute0` for the label that holds the designer,
margin or season dimension. Results reflect product attributes at the time of
the click.

## Q8. Negatives and brand lists

Q8a: campaign level.

```sql
SELECT campaign.id, campaign_criterion.type, campaign_criterion.negative,
       campaign_criterion.keyword.text, campaign_criterion.keyword.match_type,
       campaign_criterion.brand_list.shared_set
FROM campaign_criterion
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND campaign_criterion.type IN ('KEYWORD', 'BRAND_LIST')
```

Q8b: ad group level.

```sql
SELECT campaign.id, ad_group.id, ad_group_criterion.negative,
       ad_group_criterion.keyword.text, ad_group_criterion.keyword.match_type
FROM ad_group_criterion
WHERE ad_group_criterion.type = 'KEYWORD'
  AND ad_group_criterion.negative = TRUE
  AND campaign.id IN ({CAMPAIGN_IDS})
```

Q8c: shared negative lists attached to the campaigns, then their terms.

```sql
SELECT campaign.id, campaign_shared_set.shared_set, campaign_shared_set.status,
       shared_set.name, shared_set.type
FROM campaign_shared_set
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND shared_set.type = 'NEGATIVE_KEYWORDS'
```

```sql
SELECT shared_set.id, shared_set.name, shared_criterion.keyword.text,
       shared_criterion.keyword.match_type
FROM shared_criterion
WHERE shared_set.type = 'NEGATIVE_KEYWORDS'
```

## Q9. Search terms (SHP-05 evidence)

```sql
SELECT campaign.id, search_term_view.search_term, search_term_view.status,
       metrics.clicks, metrics.cost_micros, metrics.conversions,
       metrics.conversions_value
FROM search_term_view
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date BETWEEN '{START_DATE}' AND '{END_DATE}'
```

## Q10. Recent changes and creations (CHG-01…03, STR-02)

```sql
SELECT change_event.change_date_time, change_event.change_resource_type,
       change_event.resource_change_operation, change_event.changed_fields,
       change_event.client_type, change_event.campaign
FROM change_event
WHERE change_event.change_date_time >= '{START_DATE}'
  AND change_event.change_date_time <= '{END_DATE}'
  AND change_event.change_resource_type IN ('CAMPAIGN', 'CAMPAIGN_BUDGET')
LIMIT 10000
```

`change_event` needs a date range within the last 30 days and a `LIMIT` of at
most 10,000, and can't return metrics. Count `CREATE` on `CAMPAIGN` for STR-02;
read `changed_fields` for budget, bid-strategy and status edits. Don't select
user email fields unless the task needs them.

## Q11. Labels

```sql
SELECT label.id, label.name, label.resource_name
FROM label
```

Match against `campaign.labels` from Q1.

## Q12. Shopping recommendations (exclusions and coverage)

```sql
SELECT recommendation.type, recommendation.campaign, recommendation.dismissed,
       recommendation.shopping_target_all_offers_recommendation,
       recommendation.shopping_add_products_to_campaign_recommendation
FROM recommendation
WHERE recommendation.type IN ('SHOPPING_TARGET_ALL_OFFERS',
                              'SHOPPING_ADD_PRODUCTS_TO_CAMPAIGN')
```

Recommendations are hints, not verdicts: an exclusion listed as intentional in
`client_rules.md` stays PASS even if Google recommends targeting all offers.
