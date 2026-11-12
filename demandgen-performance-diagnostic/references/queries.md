# Read-only GAQL queries (Google Ads API v24 field names)

All queries are reads (`search` tool). Placeholders: `{CID}`, `{CAMPAIGN_IDS}`
(comma-separated), `{CAMPAIGN_ID}` (one campaign, Q15), `{PRIOR_START}`,
`{PRIOR_END}`, `{CURRENT_START}`, `{CURRENT_END}` (the windows from step 3),
`{BASE_START}`/`{BASE_END}` (the last 30 days before the lag cut-off),
`{START_DATE}`/`{END_DATE}` (change history, within the last 30 days). Dates are
`YYYY-MM-DD`. Run `get_resource_metadata` first and drop any field your version
rejects. Divide `*_micros` by 1,000,000.

## Q0. Enabled client accounts (if no customer ID was given)

```sql
SELECT customer_client.id, customer_client.descriptive_name,
       customer_client.status, customer_client.manager
FROM customer_client
WHERE customer_client.status = 'ENABLED' AND customer_client.manager = FALSE
```

## Q1. Demand Gen scope and daily series (no status filter, so paused/removed campaigns that spent are included)

```sql
SELECT campaign.id, campaign.name, campaign.status, campaign.primary_status,
       campaign.primary_status_reasons, campaign.bidding_strategy_type,
       campaign.bidding_strategy_system_status, campaign.start_date_time,
       campaign.end_date_time, campaign_budget.amount_micros, segments.date,
       metrics.impressions, metrics.clicks, metrics.cost_micros,
       metrics.conversions, metrics.conversions_value,
       metrics.view_through_conversions
FROM campaign
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND segments.date BETWEEN '{PRIOR_START}' AND '{CURRENT_END}'
  AND metrics.cost_micros > 0
```

## Q1b. Bidding, budget and feed settings

```sql
SELECT campaign.id, campaign.bidding_strategy_type, campaign.bidding_strategy,
       campaign.target_spend.cpc_bid_ceiling_micros,
       campaign.target_cpc.target_cpc_micros,
       campaign.maximize_conversions.target_cpa_micros,
       campaign.target_cpa.target_cpa_micros,
       campaign.maximize_conversion_value.target_roas,
       campaign.target_roas.target_roas,
       campaign.view_through_conversion_optimization_enabled,
       campaign.demand_gen_campaign_settings.upgraded_targeting,
       campaign.shopping_setting.merchant_id,
       campaign.shopping_setting.feed_label,
       campaign.shopping_setting.disable_product_feed,
       campaign_budget.amount_micros, campaign_budget.has_recommended_budget,
       campaign_budget.recommended_budget_amount_micros
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
```

## Q2. Value per conversion across comparable campaigns (MEAS-01), mature days only

```sql
SELECT campaign.id, campaign.name, campaign.advertising_channel_type,
       metrics.cost_micros, metrics.conversions, metrics.conversions_value
FROM campaign
WHERE segments.date BETWEEN '{BASE_START}' AND '{BASE_END}'
  AND metrics.conversions > 0
```

## Q3. Conversion goals and actions (MEA-01, MEA-02, MEA-05)

```sql
SELECT campaign.id, campaign_conversion_goal.category,
       campaign_conversion_goal.origin, campaign_conversion_goal.biddable
FROM campaign_conversion_goal
WHERE campaign.id IN ({CAMPAIGN_IDS})
```

```sql
SELECT conversion_action.id, conversion_action.name, conversion_action.status,
       conversion_action.type, conversion_action.category,
       conversion_action.primary_for_goal,
       conversion_action.include_in_conversions_metric,
       conversion_action.counting_type,
       conversion_action.click_through_lookback_window_days,
       conversion_action.view_through_lookback_window_days,
       conversion_action.value_settings.default_value,
       conversion_action.value_settings.always_use_default_value,
       conversion_action.attribution_model_settings.attribution_model
FROM conversion_action
WHERE conversion_action.status = 'ENABLED'
```

## Q4. Conversions by action (no cost: it can't be combined with the conversion-action segment)

```sql
SELECT campaign.id, segments.date, segments.conversion_action_name,
       segments.conversion_action_category, metrics.conversions,
       metrics.conversions_value, metrics.all_conversions,
       metrics.all_conversions_value
FROM campaign
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND segments.date BETWEEN '{PRIOR_START}' AND '{CURRENT_END}'
```

## Q5. Click / engaged-view / view-through split (DG-11) and conversion lag (CHG-04)

```sql
SELECT campaign.id, segments.conversion_attribution_event_type,
       metrics.conversions, metrics.conversions_value,
       metrics.all_conversions, metrics.all_conversions_value
FROM campaign
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND segments.date BETWEEN '{PRIOR_START}' AND '{CURRENT_END}'
```

```sql
SELECT campaign.id, segments.conversion_lag_bucket,
       metrics.conversions, metrics.conversions_value
FROM campaign
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND segments.date DURING LAST_90_DAYS
```

Pair Q5 with `metrics.view_through_conversions` from Q1. If your version rejects
a metric with the attribution segment, check `selectable_with` in
`get_resource_metadata` and drop it.

## Q6. Offline upload health

```sql
SELECT offline_conversion_upload_conversion_action_summary.conversion_action_name,
       offline_conversion_upload_conversion_action_summary.successful_event_count,
       offline_conversion_upload_conversion_action_summary.total_event_count,
       offline_conversion_upload_conversion_action_summary.status
FROM offline_conversion_upload_conversion_action_summary
```

No rows = no offline uploads; say so and continue.

## Q7. Portfolio bid strategies (when `campaign.bidding_strategy` is set)

```sql
SELECT bidding_strategy.id, bidding_strategy.name, bidding_strategy.type,
       bidding_strategy.maximize_conversion_value.target_roas,
       bidding_strategy.maximize_conversions.target_cpa_micros,
       bidding_strategy.target_roas.target_roas,
       bidding_strategy.target_cpa.target_cpa_micros
FROM bidding_strategy
```

## Q8. Change history (≤ 30 days back, LIMIT ≤ 10000, no metrics)

```sql
SELECT change_event.change_date_time, change_event.change_resource_type,
       change_event.change_resource_name, change_event.resource_change_operation,
       change_event.changed_fields, change_event.client_type,
       change_event.old_resource, change_event.new_resource,
       change_event.campaign, change_event.ad_group
FROM change_event
WHERE change_event.change_date_time >= '{START_DATE}'
  AND change_event.change_date_time <= '{END_DATE}'
  AND change_event.campaign_advertising_channel_type = 'DEMAND_GEN'
ORDER BY change_event.change_date_time
LIMIT 10000
```

## Q9. Channel controls and optimized targeting

```sql
SELECT campaign.id, ad_group.id, ad_group.name, ad_group.status,
       ad_group.optimized_targeting_enabled,
       ad_group.demand_gen_ad_group_settings.channel_controls.channel_config,
       ad_group.demand_gen_ad_group_settings.channel_controls.channel_strategy,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.youtube_in_stream,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.youtube_in_feed,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.youtube_shorts,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.discover,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.gmail,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.display,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.maps
FROM ad_group
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND ad_group.status != 'REMOVED'
```

`selected_channels.maps` is new in v24; drop it if rejected.

## Q10. Channel mix per window

```sql
SELECT campaign.id, segments.ad_network_type, metrics.impressions,
       metrics.clicks, metrics.cost_micros, metrics.conversions,
       metrics.conversions_value, metrics.view_through_conversions
FROM campaign
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND segments.date BETWEEN '{PRIOR_START}' AND '{CURRENT_END}'
```

YouTube surfaces: add `segments.ad_sub_network_type` (must be selected with
`segments.ad_network_type`).

## Q11. Ads, policy and asset labels

```sql
SELECT campaign.id, ad_group.id, ad_group_ad.ad.id, ad_group_ad.ad.type,
       ad_group_ad.status, ad_group_ad.primary_status,
       ad_group_ad.primary_status_reasons,
       ad_group_ad.policy_summary.approval_status, ad_group_ad.ad_strength
FROM ad_group_ad
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND ad_group_ad.status != 'REMOVED'
```

Q11b. Asset labels:

```sql
SELECT ad_group.id, ad_group_ad_asset_view.ad_group_ad,
       ad_group_ad_asset_view.asset, ad_group_ad_asset_view.field_type,
       ad_group_ad_asset_view.performance_label,
       ad_group_ad_asset_view.enabled
FROM ad_group_ad_asset_view
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
```

Q11c. Results by ad type per window:

```sql
SELECT campaign.id, ad_group_ad.ad.type, metrics.impressions, metrics.clicks,
       metrics.cost_micros, metrics.conversions, metrics.conversions_value
FROM ad_group_ad
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND segments.date BETWEEN '{PRIOR_START}' AND '{CURRENT_END}'
```

## Q12. Seasonality adjustments and data exclusions

```sql
SELECT bidding_seasonality_adjustment.name, bidding_seasonality_adjustment.scope,
       bidding_seasonality_adjustment.status,
       bidding_seasonality_adjustment.start_date_time,
       bidding_seasonality_adjustment.end_date_time,
       bidding_seasonality_adjustment.conversion_rate_modifier,
       bidding_seasonality_adjustment.advertising_channel_types,
       bidding_seasonality_adjustment.campaigns
FROM bidding_seasonality_adjustment
```

```sql
SELECT bidding_data_exclusion.name, bidding_data_exclusion.scope,
       bidding_data_exclusion.status, bidding_data_exclusion.start_date_time,
       bidding_data_exclusion.end_date_time,
       bidding_data_exclusion.advertising_channel_types,
       bidding_data_exclusion.campaigns
FROM bidding_data_exclusion
```

## Q13. Video engagement

```sql
SELECT campaign.id, ad_group.id, metrics.impressions,
       metrics.video_trueview_views, metrics.video_trueview_view_rate,
       metrics.video_trueview_view_rate_in_stream,
       metrics.video_trueview_view_rate_in_feed,
       metrics.video_trueview_view_rate_shorts, metrics.trueview_average_cpv,
       metrics.video_quartile_p25_rate, metrics.video_quartile_p50_rate,
       metrics.video_quartile_p75_rate, metrics.video_quartile_p100_rate,
       metrics.average_video_watch_time_duration_millis
FROM ad_group
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND segments.date BETWEEN '{PRIOR_START}' AND '{CURRENT_END}'
```

## Q14. Audiences

```sql
SELECT campaign.id, ad_group.id, ad_group_criterion.type,
       ad_group_criterion.negative, ad_group_criterion.audience.audience,
       ad_group_criterion.user_list.user_list
FROM ad_group_criterion
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND ad_group_criterion.type IN ('AUDIENCE', 'USER_LIST')
```

```sql
SELECT user_list.id, user_list.name, user_list.type,
       user_list.size_for_display,
       user_list.lookalike_user_list.seed_user_list_ids,
       user_list.lookalike_user_list.expansion_level
FROM user_list
WHERE user_list.type = 'LOOKALIKE'
```

## Q15. Products in a Demand Gen campaign (campaign scope)

```sql
SELECT shopping_product.item_id, shopping_product.title,
       shopping_product.feed_label, shopping_product.status,
       shopping_product.issues, metrics.impressions, metrics.clicks,
       metrics.ctr
FROM shopping_product
WHERE shopping_product.campaign = 'customers/{CID}/campaigns/{CAMPAIGN_ID}'
  AND segments.date BETWEEN '{CURRENT_START}' AND '{CURRENT_END}'
```

Campaign scope needs the **equality filter on the campaign**; without it you get
every product in the linked Merchant Center accounts. In Demand Gen scope only
`impressions`, `clicks` and `ctr` are supported, and date segmentation
(selecting `segments.date`) isn't allowed; filtering on it is.
