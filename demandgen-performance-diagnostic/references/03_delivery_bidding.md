# 3. Delivery, bidding and budget

Search impression-share metrics don't apply to Demand Gen, so judge limited
delivery by reading statuses, daily pacing against budget, targets and inventory
together.

Check              | What to verify                                                      | Data                                                                                                                                                                                                                                                                                                                                                          | How to judge
------------------ | ------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Status             | Why the campaign is limited                                         | Q1: `campaign.primary_status`, `campaign.primary_status_reasons`, `campaign.bidding_strategy_system_status`                                                                                                                                                                                                                                                   | Map to a branch (table below). `NOT_ELIGIBLE` or `HAS_ADS_DISAPPROVED` on every ad group = BLOCK (nothing can serve)
Bid strategy       | Which strategy, and does it fit the goal                            | Q1b: `campaign.bidding_strategy_type`; `campaign.target_spend.cpc_bid_ceiling_micros`; `campaign.target_cpc.target_cpc_micros`; `campaign.maximize_conversions.target_cpa_micros`; `campaign.target_cpa.target_cpa_micros`; `campaign.maximize_conversion_value.target_roas`; `campaign.target_roas.target_roas`; portfolio: `campaign.bidding_strategy` → Q7 | See "Bidding branches" below
BID-01 / DG-01     | Bidding fits the goal; value-based when purchase values are tracked | Strategy type + MEA-02 result                                                                                                                                                                                                                                                                                                                                 | WARN if `MAXIMIZE_CONVERSIONS`/`TARGET_CPA` with reliable purchase values, or click bidding (`TARGET_SPEND`, `TARGET_CPC`) on a sales goal
BID-02             | Target vs history                                                   | tCPA / tROAS vs trailing 30-day actual of the campaign (or comparable non-brand campaigns)                                                                                                                                                                                                                                                                    | Per BID-02 bands. A target in the WARN or BLOCK band with spend far below budget is "Target-led"
BUD-01 / DG-02     | Budget headroom for conversion or value bidding                     | `campaign_budget.amount_micros` vs tCPA (or 30-day cost per conversion)                                                                                                                                                                                                                                                                                       | BUD-01 is the floor; DG-02 is the Demand Gen-specific multiple. Below either: "Budget-led / starved learning"
Pacing             | Spend vs budget per day                                             | Q1 daily rows: `metrics.cost_micros` vs `campaign_budget.amount_micros`                                                                                                                                                                                                                                                                                       | Spend at budget most days + `BUDGET_CONSTRAINED` = Budget-led. Spend well under budget with a strict target = Target-led. Under budget with no target = Inventory/creative/audience-led (go to step 7)
DG-03 / MEA-06     | Enough conversions to judge efficiency                              | Primary conversions since launch or the last structural change                                                                                                                                                                                                                                                                                                | Per DG-03 (conversion floor plus CHG-03/CHG-04). Below it: "too early": report the trend, give no efficiency verdict and no structural recommendation
Recommended budget | Google's own budget signal                                          | `campaign_budget.has_recommended_budget`, `recommended_budget_amount_micros`                                                                                                                                                                                                                                                                                  | Context only. Any increase follows CHG-01 steps

## Status reason → branch

`primary_status_reasons` / system status                                                                                            | Branch                      | Next
----------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | ----
`BUDGET_CONSTRAINED`, `LIMITED_BY_BUDGET`                                                                                           | Budget-led                  | If efficiency is at or better than target, suggest CHG-01 budget steps; otherwise fix efficiency first
`BIDDING_STRATEGY_LIMITED`, `BIDDING_STRATEGY_CONSTRAINED`, `LIMITED_BY_INVENTORY`, `LIMITED_BY_DATA`, `LIMITED_BY_CPC_BID_CEILING` | Target-led or inventory-led | Compare target with history (BID-02); check channels, audiences, creative formats (step 7)
`BIDDING_STRATEGY_LEARNING`, `LEARNING_*`                                                                                           | Learning                    | Go to step 6: which change triggered it? No new edits until it settles (CHG-03)
`BIDDING_STRATEGY_MISCONFIGURED`, `MISCONFIGURED_*`                                                                                 | Tracking / setup            | Usually a conversion goal problem: back to step 4
`HAS_ADS_DISAPPROVED`, `HAS_ADS_LIMITED_BY_POLICY`, `MOST_ADS_UNDER_REVIEW`                                                         | Creative / policy           | Step 7 (ad policy)
`CAMPAIGN_PAUSED`, `CAMPAIGN_ENDED`, `CAMPAIGN_REMOVED`                                                                             | Not serving                 | Find when (change_event, last day with cost) and whether it explains the drop

## Bidding branches

-   **Maximize clicks (`TARGET_SPEND`) or Target CPC (`TARGET_CPC`).** The
    system buys clicks, not sales. Expect low value per \$1 and many
    `INTERACTION` conversions of low value. Diagnose it as "strategy fit", not
    as a creative or audience failure. If purchases are tracked with values and
    DG-03 volume exists (in this or comparable campaigns), suggest a value-based
    strategy via a campaign experiment (CHG-05), not a mid-flight switch. A very
    low CPC ceiling (`cpc_bid_ceiling_micros`) also restricts inventory to the
    cheapest placements.
-   **Maximize conversions / tCPA.** Check the target (BID-02), the budget
    (DG-02) and that the biddable action is a purchase (MEA-01). If purchase
    values vary a lot by product, value-based bidding is the better fit
    (BID-01).
-   **Maximize conversion value / tROAS.** Needs real values (MEA-02) and enough
    volume. A tROAS far above history under-serves (BID-02). A sudden value drop
    with stable conversions points to values (MEAS-01), a product mix change, or
    a `conversion_action` value setting change.

## Calculations

-   ROAS = conversion value / cost; value per \$1 is the same number.
-   Cost per conversion = cost / conversions, same actions and window as the
    bidding goal. State whether engaged-view conversions are included.
-   Budget multiple = daily budget / target CPA (or 30-day cost per conversion).
