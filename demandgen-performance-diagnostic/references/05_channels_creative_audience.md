# 5. Channels, creative and audiences

Run this when delivery is limited without a budget or target cause, or when the
drop sits in one channel or one conversion type.

## Channels

Check               | What to verify                                      | Data                                                                                                                                                                                                                                                                                                            | How to judge
------------------- | --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Controls            | Which channels each ad group may use                | Q9: `ad_group.demand_gen_ad_group_settings.channel_controls.channel_config` (`CHANNEL_STRATEGY` or `SELECTED_CHANNELS`), `.channel_strategy` (`ALL_CHANNELS`, `ALL_OWNED_AND_OPERATED_CHANNELS`), `.selected_channels.youtube_in_stream / youtube_in_feed / youtube_shorts / discover / gmail / display / maps` | Hand-picked channels reduce reach. Suggest all channels unless there's a written brand or measurement reason. `maps` was added in v24.2; check it exists in your version
Mix                 | Where spend and conversions went in each window     | Q10: `segments.ad_network_type` (`YOUTUBE`, `DISCOVER`, `GMAIL`, `CONTENT`, `MAPS`, `GOOGLE_TV`); for YouTube add `segments.ad_sub_network_type` (`YOUTUBE_INSTREAM`, `YOUTUBE_INFEED`, `YOUTUBE_SHORTS`)                                                                                                       | Find which channel's cost or conversions moved. `GOOGLE_OWNED_CHANNELS` appears only in older data
Selected but silent | A channel is allowed but gets almost no impressions | Q9 vs Q10                                                                                                                                                                                                                                                                                                       | Usually a missing format (e.g. no vertical video for Shorts, no images for Discover/Gmail). Inferred; confirm in step "Creative"

## Creative

Check             | What to verify                      | Data                                                                                                                                                                                                                         | How to judge
----------------- | ----------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Policy            | Ads that can't serve fully          | Q11: `ad_group_ad.policy_summary.approval_status`, `ad_group_ad.primary_status`, `ad_group_ad.primary_status_reasons`, `ad_group_ad.ad.type`                                                                                 | `DISAPPROVED` on all ads of an ad group that should serve = BLOCK for that ad group; `APPROVED_LIMITED` = WARN (AST-08 tiers)
Ad types          | Format coverage per ad group        | `ad_group_ad.ad.type` (`DEMAND_GEN_MULTI_ASSET_AD`, `DEMAND_GEN_VIDEO_RESPONSIVE_AD`, `DEMAND_GEN_CAROUSEL_AD`, `DEMAND_GEN_PRODUCT_AD`)                                                                                     | An ad group with images only can't earn engaged-view conversions; video only can't serve image placements well
Asset coverage    | Image ratios and video orientations | Q11b: `ad_group_ad_asset_view.field_type` (`MARKETING_IMAGE` 1.91:1, `SQUARE_MARKETING_IMAGE` 1:1, `PORTRAIT_MARKETING_IMAGE` 4:5, `TALL_PORTRAIT_MARKETING_IMAGE` 9:16, `YOUTUBE_VIDEO`, `LOGO`, `HEADLINE`, `DESCRIPTION`) | Missing ratios or no vertical video limit placements. Use the checks in `../demandgen-setup-audit/SKILL.md` for exact counts
Asset performance | Labels per asset                    | `ad_group_ad_asset_view.performance_label` (`BEST`, `GOOD`, `LOW`, `LEARNING`, `PENDING`), `.enabled`                                                                                                                        | Flag ad groups where most enabled assets are `LOW`. Replace a few at a time; swapping everything at once restarts learning (CHG-03)
Video engagement  | Do people watch?                    | Q13: `metrics.video_trueview_view_rate`, `_in_stream`, `_in_feed`, `_shorts`, `metrics.video_quartile_p25_rate` … `p100_rate`, `metrics.average_video_watch_time_duration_millis`, `metrics.trueview_average_cpv`            | Compare with the account's own history and other ad groups, not external benchmarks. If most viewers leave before the midpoint, the opening holds attention but the rest of the video doesn't (inferred). Healthy views with few engaged-view conversions point to the landing page or offer

## Audiences

Check               | What to verify                                                 | Data                                                                                                                                                                                                  | How to judge
------------------- | -------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Signals             | Audiences per ad group                                         | Q14: `ad_group_criterion.type = 'AUDIENCE'`, `ad_group_criterion.audience.audience`, `ad_group_criterion.user_list.user_list`; `user_list.lookalike_user_list.seed_user_list_ids`, `.expansion_level` | Flag ad groups with no first-party signal (buyers, site visitors) where lists exist. Lookalikes need a reasonable seed; check `user_list.size_for_display` of the seed
Optimized targeting | Is it on?                                                      | `ad_group.optimized_targeting_enabled`                                                                                                                                                                | Off + narrow audience + target-led underspend = Audience-led (inferred). On = audiences are signals, so a "wrong audience" theory is weaker
Overlap             | Several ad groups or campaigns on the same audience and market | Q14 across campaigns                                                                                                                                                                                  | Splits learning (STR-01). Suggest consolidation, not more ad groups

## Notes for multi-market ecommerce advertisers

-   Print-on-demand creative: show the product in use (shirt worn, mug on a
    desk), personalisation, and occasion (gift, Christmas). Product-only
    white-background images suit product ads, less so video placements.
-   Seasonal campaigns need seasonal creative; a Christmas campaign running
    generic store videos in November is a likely creative cause (inferred).
