# 2. Channels and targeting

Demand Gen can serve on YouTube (In-Stream, In-Feed, Shorts), Discover, Gmail,
Google Display Network and Maps. Channel controls are set per ad group.

Check   | What to verify                         | Data                                                                                                                                                                                                        | How to judge
------- | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
DG-04   | Channel controls                       | `ad_group.demand_gen_ad_group_settings.channel_controls.channel_config`; `.channel_strategy`; `.selected_channels.youtube_in_stream / youtube_in_feed / youtube_shorts / discover / gmail / display / maps` | PASS: `channel_strategy = ALL_CHANNELS`, or a restriction with a recorded reason. WARN: `ALL_OWNED_AND_OPERATED_CHANNELS` or `SELECTED_CHANNELS` with no reason
SRCH-13 | Location and language match the market | Campaign or ad group `LOCATION` / `LANGUAGE` criteria; `campaign.geo_target_type_setting.positive_geo_target_type`; markets the store ships to                                                              | PASS: `PRESENCE` and the language of the ads. WARN: `PRESENCE_OR_INTEREST` (or `SEARCH_INTEREST`) with no reason, or a language mismatch. BLOCK: a country the store/feed doesn't serve

**Reading channel controls**

-   `channel_config = CHANNEL_STRATEGY`: read `channel_strategy` (`ALL_CHANNELS`
    or `ALL_OWNED_AND_OPERATED_CHANNELS`, which drops third-party Display
    inventory).
-   `channel_config = SELECTED_CHANNELS`: read each `selected_channels.*`
    boolean. At least one must be true.
-   Empty or `UNSPECIFIED` in an older campaign: treat as all channels, and
    confirm in the UI if it matters.

**Good reasons to restrict** (record the reason in the plan)

-   A test that needs one surface (e.g. Shorts-only), set up as its own ad group
    with its own creative.
-   Brand-safety requirements that rule out third-party inventory.
-   Creative that only fits one surface. Fix the creative first (DG-06) rather
    than restricting.

**Targeting levels.** New Demand Gen campaigns use ad group-level location and
language targeting (`campaign.demand_gen_campaign_settings.upgraded_targeting`,
set at creation). Check criteria at both levels: `campaign_criterion` and
`ad_group_criterion` with `type IN ('LOCATION', 'LANGUAGE')`.

**Common pattern**

-   Each market has its own language and currency. An ad group that targets
    Germany should show German ads and land on the German store (see SRCH-09 in
    [search-setup-audit](../../search-setup-audit/references/05_landing_targeting.md)
    for landing-page checks).
-   Use `PRESENCE` for markets the store ships to, so the budget isn't spent on
    people outside the market who only show interest in it.
