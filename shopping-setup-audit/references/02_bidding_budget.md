# 2. Bidding, CPC caps and budget

Check  | What to verify                                       | Data                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | How to judge
------ | ---------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
SHP-01 | Bid strategy fits the goal                           | `campaign.bidding_strategy_type` (or the portfolio `bidding_strategy.type`), MEA-01/02 results, the plan's stated reason and exit criteria                                                                                                                                                                                                                                                                                                                                                                                                                                               | PASS: `MAXIMIZE_CONVERSION_VALUE` or `TARGET_ROAS` with purchase values tracked. WARN: `TARGET_SPEND` (Maximize clicks), `MANUAL_CPC` or `MAXIMIZE_CONVERSIONS`, with a written reason, an end date within the SHP-01 window and exit criteria. BLOCK: new campaigns in a plan on click-based bidding with no exit criteria
SHP-02 | CPC caps and bid limits aren't starving the campaign | Click-based: `campaign.target_spend.cpc_bid_ceiling_micros` (or portfolio `bidding_strategy.target_spend.cpc_bid_ceiling_micros`), product-group `ad_group_criterion.cpc_bid_micros` / `effective_cpc_bid_micros`. Value-based: `campaign.maximize_conversion_value.cpc_bid_ceiling_micros` or `campaign.target_roas.cpc_bid_ceiling_micros`, or the portfolio's `bidding_strategy.maximize_conversion_value.cpc_bid_ceiling_micros` / `bidding_strategy.target_roas.cpc_bid_ceiling_micros`. Compare with the 30-day `metrics.average_cpc` of comparable Shopping/PMax traffic (Q2, Q7) | Use the SHP-02 bands (cap ÷ comparable average CPC). Any bid limit on a value-based strategy is WARN unless the plan records why
BID-01 | Value-based bidding for ecommerce                    | Same as SHP-01                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | WARN if `MAXIMIZE_CONVERSIONS` is used while purchase values are tracked
BID-02 | Starting tROAS anchored to history                   | Planned tROAS vs the 30-day ROAS of comparable **non-brand** campaigns (same country and product range)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  | PASS within ±20%; WARN 20–50% stricter; BLOCK > 50% stricter. Below MEA-06 volume, suggest no target at first
BUD-01 | Budget headroom                                      | `campaign_budget.amount_micros` vs target CPA or the comparable 30-day cost per conversion                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | WARN if < 3×. Note that ≥ 10× gives a faster ramp-up. For a campaign meant to learn, a budget that can't buy BUD-01 headroom is a sign it should be merged
CHG-01 | Budget change step (edits)                           | Old vs new budget; last budget change from `change_event` (Q10)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          | ≤ 20% and ≥ 3 days since the last change = PASS; 20–50% or < 3 days = WARN; > 50% = BLOCK without approval at the EXE-05 tier
CHG-02 | Target change step (edits)                           | Old vs new tROAS; last target change                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     | ≤ 15% and ≥ 7 days = PASS; 15–30% or < 7 days = WARN; > 30% = BLOCK without approval

**Why tiny caps fail (explain this in SHP-02 findings)**

-   A cap far below what comparable Shopping traffic costs means the campaign
    only wins auctions nobody else wants. It may spend, but the clicks tend to
    convert poorly, and the low CPC hides a low value per \$1.
-   Compare value per \$1 (conversion value / cost) with the value-based
    baseline over the same mature window, not CPC or clicks.

**Compliant alternatives to offer**

1.  **Recommended:** put the products into one existing or new value-based
    campaign for that country (Maximize conversion value; add a tROAS only when
    MEA-06 passes), segmented by product groups.
2.  Keep click-based bidding as a time-boxed test: cap at least at the SHP-02
    PASS level, a written end date, and an exit rule (switch to value-based at
    MEA-06 volume, or stop if value per \$1 stays below the baseline after
    CHG-04/DX-03).
3.  If budget is the worry, use a smaller daily budget rather than a tiny cap.

**Calculations**

-   ROAS = conversion value / cost; "stricter tROAS" = a higher target.
-   Average CPC from `metrics.average_cpc` (micros) or cost / clicks, over the
    same window and country.
-   Budget changes: count every change to the same budget in `change_event`, by
    any client type, in the last 14 days.

**The advertiser note:** two budget changes less than 3 days apart on the same
campaign is the most common CHG-01 finding. Batch budget edits into one reviewed
step and wait at least the CHG-01 interval.
