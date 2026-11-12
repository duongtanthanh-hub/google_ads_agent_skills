# 2. Delivery and impression share (DX-04 step 2)

Can the campaign serve, and is it losing auctions to budget or to rank?

Check           | What to verify                         | Data                                                                                                                                                  | How to judge
--------------- | -------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Status          | Campaign primary status and reasons    | Q1 `campaign.primary_status`, `campaign.primary_status_reasons`                                                                                       | BLOCK if `NOT_ELIGIBLE` or `MISCONFIGURED`. `LIMITED` / `LEARNING`: read the reasons (`BUDGET_CONSTRAINED`, `BIDDING_STRATEGY_LIMITED`, `BIDDING_STRATEGY_LEARNING`, `HAS_ADS_DISAPPROVED`, `AD_GROUPS_PAUSED`, …)
Ad groups       | Shopping ad groups enabled and serving | Q1b `ad_group.status`, `ad_group.primary_status`, `ad_group.type = SHOPPING_PRODUCT_ADS`                                                              | WARN if ad groups that carried base-window cost are paused or limited
DXP-02          | Spend vs budget                        | Q2 daily cost vs `campaign_budget.amount_micros`                                                                                                      | Bands per DXP-02. On a click strategy with a cap, underspend usually points to the cap (step 3), not demand
BUD-01          | Budget headroom                        | Budget vs 30-day cost per conversion                                                                                                                  | Bands per BUD-01 (only meaningful when the campaign has conversions)
DXP-03          | Impression share split                 | Q10 at campaign level (also ad group and product group levels, see below)                                                                             | Bands per DXP-03. Budget-lost high → budget-limited; rank-lost high → bid/cap or quality-limited
Click share     | Share of achievable clicks             | Q10 `metrics.search_click_share`                                                                                                                      | Low click share with high rank-lost share supports a bid/cap cause (DXP-06)
Prominence      | Absolute top share                     | Q10 `metrics.search_absolute_top_impression_share`                                                                                                    | Context only; a drop with stable IS means you're showing in less prominent positions
FEED-01         | Products available for the feed label  | Q1 `shopping_setting.feed_label`, Q7 statuses                                                                                                         | BLOCK if no eligible products
Merchant Center | Account suspended or warned            | Q13 recommendation types `SHOPPING_FIX_SUSPENDED_MERCHANT_CENTER_ACCOUNT`, `SHOPPING_FIX_MERCHANT_CENTER_ACCOUNT_SUSPENSION_WARNING`; Merchant Center | BLOCK if suspended (nothing can serve); WARN on a warning

**Which impression-share metrics exist for Shopping** Documented for Shopping
campaigns (Google Search inventory only): `search_impression_share`,
`search_budget_lost_impression_share`, `search_rank_lost_impression_share`,
`search_absolute_top_impression_share`, `search_click_share`. They're available
at campaign, ad group and product group level in the UI. In the API, query them
on `campaign`, `ad_group` or `product_group_view`; for
`shopping_performance_view` (product ID level), check with
`get_resource_metadata` first. `search_exact_match_impression_share` is
keyword-based; don't use it for Shopping.

**Decision path for "few clicks / underspend"**

1.  Not serving or misconfigured → BLOCK, fix first.
2.  Budget-lost share high and the budget runs out → budget-limited; a budget
    step (CHG-01) is valid only if value per \$1 meets the goal.
3.  Rank-lost share high, with a cap or low manual bids → DXP-06 (step 3).
4.  Both low but impressions fell → fewer eligible products (step 5), traffic
    moved to an overlapping campaign (step 6), or demand (step 7).

**Gotchas**

-   IS below 0.1 comes back as 0.0999; lost share above 0.9 as 0.9001.
-   Low-traffic rows can return no IS at all; mark NOT_ASSESSED rather than 0.
-   One day above budget isn't a fault (daily spend can reach 2× the average
    daily budget; monthly charges are capped at 30.4×).
