# Read-only GAQL queries (Google Ads API v24 field names)

All queries are reads. Replace `{CAMPAIGN_IDS}` with a comma-separated list of
IDs. Check unfamiliar fields with `get_resource_metadata` first, and use
`SearchStream` for large results. Money fields in micros: divide by 1,000,000.

## Q0. Accessible, enabled client accounts (pre-flight)

Call `list_accessible_customers` first, then under each returned customer:

```sql
SELECT customer_client.id, customer_client.descriptive_name,
       customer_client.status, customer_client.manager
FROM customer_client
WHERE customer_client.status = 'ENABLED' AND customer_client.manager = FALSE
```

Run every later query only against these client IDs.

## Q1. Campaign settings

```sql
SELECT campaign.id, campaign.name, campaign.status, campaign.primary_status,
       campaign.primary_status_reasons, campaign.bidding_strategy_type,
       campaign.bidding_strategy_system_status,
       campaign.maximize_conversions.target_cpa_micros,
       campaign.maximize_conversion_value.target_roas,
       campaign.target_cpa.target_cpa_micros, campaign.target_roas.target_roas,
       campaign_budget.amount_micros,
       campaign.shopping_setting.merchant_id,
       campaign.shopping_setting.feed_label,
       campaign.shopping_setting.disable_product_feed,
       campaign.demand_gen_campaign_settings.upgraded_targeting,
       campaign.geo_target_type_setting.positive_geo_target_type,
       campaign.view_through_conversion_optimization_enabled,
       campaign.start_date_time
FROM campaign
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND campaign.id IN ({CAMPAIGN_IDS})
```

## Q2. 30-day baseline (comparable campaigns; exclude brand Search)

```sql
SELECT campaign.id, campaign.name, campaign.advertising_channel_type,
       metrics.cost_micros, metrics.conversions, metrics.conversions_value
FROM campaign
WHERE segments.date DURING LAST_30_DAYS
  AND campaign.status = 'ENABLED'
  AND metrics.cost_micros > 0
```

## Q3. Ad groups: channel controls, optimized targeting, target overrides

```sql
SELECT campaign.id, ad_group.id, ad_group.name, ad_group.status,
       ad_group.primary_status,
       ad_group.demand_gen_ad_group_settings.channel_controls.channel_config,
       ad_group.demand_gen_ad_group_settings.channel_controls.channel_strategy,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.youtube_in_stream,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.youtube_in_feed,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.youtube_shorts,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.discover,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.gmail,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.display,
       ad_group.demand_gen_ad_group_settings.channel_controls.selected_channels.maps,
       ad_group.optimized_targeting_enabled,
       ad_group.exclude_demographic_expansion,
       ad_group.target_cpa_micros, ad_group.target_roas,
       ad_group.effective_target_cpa_micros, ad_group.effective_target_roas
FROM ad_group
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND ad_group.status != 'REMOVED'
```

If the API rejects a deep `selected_channels` path, select
`ad_group.demand_gen_ad_group_settings.channel_controls.channel_config` and
`.channel_strategy` first, then add the channel booleans one by one after
checking them with `get_resource_metadata`.

## Q4. Ads and their assets

```sql
SELECT ad_group.id, ad_group_ad.ad.id, ad_group_ad.ad.type,
       ad_group_ad.status, ad_group_ad.primary_status,
       ad_group_ad.ad_strength, ad_group_ad.policy_summary.approval_status,
       ad_group_ad.ad.final_urls,
       ad_group_ad.ad.demand_gen_multi_asset_ad.headlines,
       ad_group_ad.ad.demand_gen_multi_asset_ad.descriptions,
       ad_group_ad.ad.demand_gen_multi_asset_ad.business_name,
       ad_group_ad.ad.demand_gen_multi_asset_ad.marketing_images,
       ad_group_ad.ad.demand_gen_multi_asset_ad.square_marketing_images,
       ad_group_ad.ad.demand_gen_multi_asset_ad.portrait_marketing_images,
       ad_group_ad.ad.demand_gen_multi_asset_ad.tall_portrait_marketing_images,
       ad_group_ad.ad.demand_gen_multi_asset_ad.logo_images,
       ad_group_ad.ad.demand_gen_video_responsive_ad.headlines,
       ad_group_ad.ad.demand_gen_video_responsive_ad.long_headlines,
       ad_group_ad.ad.demand_gen_video_responsive_ad.descriptions,
       ad_group_ad.ad.demand_gen_video_responsive_ad.videos,
       ad_group_ad.ad.demand_gen_video_responsive_ad.logo_images,
       ad_group_ad.ad.demand_gen_video_responsive_ad.business_name,
       ad_group_ad.ad.demand_gen_carousel_ad.carousel_cards,
       ad_group_ad.ad_group_ad_asset_automation_settings
FROM ad_group_ad
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND ad_group_ad.status != 'REMOVED'
```

Image and video lists hold asset resource names. Resolve them with Q5.

## Q5. Image sizes and video orientation

```sql
SELECT asset.resource_name, asset.id, asset.type, asset.orientation,
       asset.image_asset.full_size.width_pixels,
       asset.image_asset.full_size.height_pixels,
       asset.youtube_video_asset.youtube_video_id
FROM asset
WHERE asset.resource_name IN ({ASSET_RESOURCE_NAMES})
```

Image ratio = width / height: 1.91 landscape, 1.0 square, 0.8 portrait (4:5),
0.5625 tall portrait (9:16), each ±1%. `asset.orientation` (`LANDSCAPE`,
`PORTRAIT`, `SQUARE`) is returned only for image and video assets; use it for
video orientation in DG-06.

## Q6. Audiences per ad group, then the lists behind them

```sql
SELECT ad_group.id, ad_group_criterion.criterion_id, ad_group_criterion.type,
       ad_group_criterion.negative, ad_group_criterion.status,
       ad_group_criterion.user_list.user_list,
       ad_group_criterion.audience.audience,
       ad_group_criterion.custom_audience.custom_audience
FROM ad_group_criterion
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND ad_group_criterion.type IN ('USER_LIST', 'AUDIENCE', 'CUSTOM_AUDIENCE')
```

```sql
SELECT user_list.id, user_list.name, user_list.type,
       user_list.size_range_for_display, user_list.size_for_display,
       user_list.eligible_for_display,
       user_list.lookalike_user_list.seed_user_list_ids,
       user_list.lookalike_user_list.expansion_level,
       user_list.lookalike_user_list.country_codes
FROM user_list
WHERE user_list.id IN ({USER_LIST_IDS})
```

Run the second query again for the seed list IDs of each lookalike.

## Q7. Product eligibility (if a feed is attached)

```sql
SELECT shopping_product.item_id, shopping_product.title,
       shopping_product.status, shopping_product.issues,
       shopping_product.feed_label, shopping_product.merchant_center_id
FROM shopping_product
WHERE shopping_product.campaign = 'customers/{CID}/campaigns/{CAMPAIGN_ID}'
```

Keep the equality filter on the campaign; without it the query returns every
product in the linked Merchant Center accounts. Confirm that `shopping_product`
covers Demand Gen campaigns in your API version; if not, check eligibility by
Merchant Center ID and feed label instead.

## Q8. Conversion goals and actions

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
       conversion_action.attribution_model_settings.attribution_model,
       conversion_action.click_through_lookback_window_days,
       conversion_action.view_through_lookback_window_days
FROM conversion_action
WHERE conversion_action.status = 'ENABLED'
```

The engaged-view conversion window isn't in these fields; read it in the UI or
mark it NOT_ASSESSED.

## Q9. Recent changes (CHG-01…03, DG-03), last 14 days

```sql
SELECT change_event.change_date_time, change_event.change_resource_type,
       change_event.resource_change_operation, change_event.changed_fields,
       change_event.client_type, change_event.campaign
FROM change_event
WHERE change_event.change_date_time >= '{START_DATE}'
  AND change_event.change_date_time <= '{END_DATE}'
  AND change_event.campaign IN ('customers/{CID}/campaigns/{CAMPAIGN_ID}')
LIMIT 10000
```

`change_event` needs a date filter within the last 30 days, a `LIMIT` of at most
10,000, and no `metrics.*`. Don't select user email fields.

## Q10. Delivery by channel (live / post_launch)

```sql
SELECT campaign.id, segments.ad_network_type, segments.ad_sub_network_type,
       metrics.impressions, metrics.clicks, metrics.cost_micros,
       metrics.conversions, metrics.conversions_value
FROM campaign
WHERE campaign.advertising_channel_type = 'DEMAND_GEN'
  AND campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date DURING LAST_30_DAYS
```

`ad_sub_network_type` (YouTube In-Stream / In-Feed / Shorts) must be selected
together with `ad_network_type`.

## Q11. Conversions by attribution event, and platform-comparable view (DG-11)

```sql
SELECT campaign.id, segments.conversion_attribution_event_type,
       metrics.conversions, metrics.conversions_value
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date DURING LAST_30_DAYS
```

```sql
SELECT campaign.id, metrics.cost_micros, metrics.conversions,
       metrics.conversions_value, metrics.platform_comparable_conversions,
       metrics.platform_comparable_conversions_value
FROM campaign
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND segments.date DURING LAST_30_DAYS
```

If a segment/metric combination is rejected, drop it and mark that part of DG-11
NOT_ASSESSED rather than guessing. Platform-comparable columns are for comparing
with social platforms; never put them next to PMax or Search
`metrics.conversions` (DG-11).

## Q12. Location and language targeting (SRCH-13)

```sql
SELECT campaign.id, campaign_criterion.type, campaign_criterion.negative,
       campaign_criterion.location.geo_target_constant,
       campaign_criterion.language.language_constant
FROM campaign_criterion
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND campaign_criterion.type IN ('LOCATION', 'LANGUAGE')
```

```sql
SELECT ad_group.id, ad_group_criterion.type, ad_group_criterion.negative,
       ad_group_criterion.location.geo_target_constant,
       ad_group_criterion.language.language_constant
FROM ad_group_criterion
WHERE campaign.id IN ({CAMPAIGN_IDS})
  AND ad_group_criterion.type IN ('LOCATION', 'LANGUAGE')
```

## Q13. Experiments on record (DG-12, CHG-05)

```sql
SELECT experiment.name, experiment.type, experiment.status,
       experiment.start_date, experiment.end_date
FROM experiment
```

Conversion Lift studies are managed in the Google Ads UI (or with the Google
account team) and may not appear here; ask for the study details if a lift study
is claimed.
