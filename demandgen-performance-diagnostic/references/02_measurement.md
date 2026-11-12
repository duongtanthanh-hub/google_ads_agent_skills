# 2. Measurement (always first)

Demand Gen can only be judged on what's measured. If this step finds a problem,
report it first and mark later findings low-confidence.

Check           | What to verify                                                                                                                    | Data                                                                                                                                                                                                                 | How to judge
--------------- | --------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
MEA-01          | Biddable goals are purchase-type; micro-conversions are secondary                                                                 | Q3: `campaign_conversion_goal.biddable`, `.category`; `conversion_action.primary_for_goal`, `.category`, `.include_in_conversions_metric`                                                                            | BLOCK if no biddable `PURCHASE` goal. WARN if `PAGE_VIEW`, `ADD_TO_CART`, `BEGIN_CHECKOUT` or `ENGAGEMENT` are biddable next to purchase. Upper-funnel formats inflate cheap micro-conversions fast
MEA-02          | Purchase actions send real transaction values                                                                                     | `conversion_action.value_settings.always_use_default_value`, `.default_value`                                                                                                                                        | WARN if a purchase action always uses the default value; BLOCK if value-based bidding relies on an action without values
MEAS-01         | Value per conversion isn't anomalous                                                                                              | Q2: `metrics.conversions_value / metrics.conversions` per campaign vs the median of comparable campaigns (same market and conversion actions, 30 mature days; comparison set and volume floor as defined in MEAS-01) | PASS / WARN / BLOCK per MEAS-01. A campaign far below the median usually counts a non-purchase action or a default value. On BLOCK, don't compare its ROAS or move budget on it until explained
Action mix      | Which actions make up the conversions                                                                                             | Q4: `segments.conversion_action_name`, `segments.conversion_action_category` (no cost in this query)                                                                                                                 | If a drop or rise comes from one action (e.g. a page-view action switched to primary, or a purchase tag broke), it's Tracking, not performance
MEA-05          | Attribution model and windows                                                                                                     | `conversion_action.attribution_model_settings.attribution_model`, `click_through_lookback_window_days`, `view_through_lookback_window_days`; `change_event` on `conversion_action`-related settings if visible       | WARN if not data-driven. A shortened window lowers reported conversions without any change in sales. The engaged-view window is set in the UI and isn't exposed in API v24: NOT_ASSESSED unless the client confirms it
Uploads         | Offline / server-side conversion pipelines                                                                                        | Q6: `offline_conversion_upload_conversion_action_summary`                                                                                                                                                            | If no rows: say no offline uploads exist and continue. Else compare `successful_event_count` with `total_event_count` and read `status` and `alerts`
MEA-03 / MEA-04 | Enhanced conversions; consent mode for EEA/UK traffic                                                                             | Client confirmation / UI (not fully in GAQL)                                                                                                                                                                         | WARN if off or unknown; NOT_ASSESSED if it can't be checked. A consent-banner change in DE/FR/IT/ES/PL can look like a Demand Gen drop
DG-11           | Conversion definition declared (actions, attribution events, windows, column) and the same as for the channels it's compared with | Q5: `segments.conversion_attribution_event_type` (`INTERACTION`, `ENGAGED_VIEW`, `IMPRESSION`); `metrics.view_through_conversions`; Q3 windows                                                                       | WARN if not declared, or if Demand Gen on one definition is compared with PMax/Search on another. This report always shows the three types as columns and states the headline basis
Warehouse check | Google Ads value vs warehouse / GA4 revenue for the same orders and days                                                          | BI warehouse                                                                                                                                                                                                         | If Ads value moves and warehouse revenue for the same orders doesn't (or vice versa), the issue is measurement or attribution, not demand

## Reading the three conversion types

Type (`conversion_attribution_event_type`) | Meaning                                                                                                                                                      | In `metrics.conversions`?               | A drop limited to this type usually means
------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------- | -----------------------------------------
`INTERACTION`                              | Conversion after a click                                                                                                                                     | Yes                                     | Click-led formats (Discover, Gmail, image, product ads) or the landing page
`ENGAGED_VIEW`                             | Conversion after an engaged view of a video ad without a click (Google: about 10 seconds watched, or the whole video if shorter; check current Help wording) | Yes, for Demand Gen                     | Less video delivery, a weaker hook, or video removed from the mix
`IMPRESSION` (view-through)                | Conversion after an impression without interaction                                                                                                           | No (`metrics.view_through_conversions`) | Display/Discover reach changes. Never added to the headline unless the client agreed

-   Report all three as columns per campaign and window. State the headline:
    e.g. "ROAS = value of click + engaged-view conversions / cost".
-   If `campaign.view_through_conversion_optimization_enabled` is true, the
    campaign also optimises toward view-through conversions. Note it, and check
    whether it changed during the windows before comparing campaigns.
-   Some API versions expose "platform-comparable" conversion metrics for
    comparison with other platforms. Use them only for that comparison, never as
    the headline, and verify them against your API version.

## Notes for multi-market ecommerce advertisers

-   Value per \$1 is the agreed efficiency metric. Before blaming Demand Gen for
    low value per \$1, rule out non-purchase actions counted as conversions
    (MEAS-01) — the same issue made some Shopping campaigns look worse than they
    were.
-   Agree with the business owner which source is the reference (Ads, GA4 or
    warehouse) and use the same one in every report.
