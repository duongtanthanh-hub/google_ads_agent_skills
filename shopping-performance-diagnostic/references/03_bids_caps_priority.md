# 3. Bids, CPC caps and campaign priority (DX-04 step 2, bid limits)

Standard Shopping gives the advertiser direct bid controls, so self-imposed
limits are a frequent cause of low delivery.

Check             | What to verify                                                        | Data                                                                                                                                                                                                                                                                                                                                                                         | How to judge
----------------- | --------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Strategy          | Bid strategy and its status                                           | Q1 `campaign.bidding_strategy_type` (`TARGET_SPEND` = Maximize clicks, `MANUAL_CPC`, `TARGET_ROAS`, `MAXIMIZE_CONVERSION_VALUE`), `campaign.bidding_strategy_system_status`                                                                                                                                                                                                  | `LIMITED_BY_CPC_BID_CEILING` = the cap binds (measured). `LIMITED_BY_BUDGET`, `LIMITED_BY_DATA`, `LIMITED_BY_INVENTORY` as named. `LEARNING_*`: DX-03
DXP-06            | CPC cap or manual bids vs market                                      | Cap: `campaign.target_spend.cpc_bid_ceiling_micros` (or the portfolio's `bidding_strategy.target_spend.cpc_bid_ceiling_micros`; for tROAS `campaign.target_roas.cpc_bid_ceiling_micros`). Manual: Q3c `ad_group_criterion.effective_cpc_bid_micros` on product groups. Market: 30-day average CPC for the same items and country in uncapped campaigns (Q6 across campaigns) | Bands per DXP-06. Name the cap, the comparison CPC, rank-lost IS and click share together
Benchmark CPC     | How others bid on similar products                                    | `metrics.benchmark_average_max_cpc` (Q3d; check the selectable resource with `get_resource_metadata`)                                                                                                                                                                                                                                                                        | Context only: a cap far below the benchmark supports DXP-06 (inferred). Not available → NOT_ASSESSED
BID-01            | Value-based bidding where purchase values are tracked                 | Q1 strategy; 30-day conversions (MEA-06)                                                                                                                                                                                                                                                                                                                                     | WARN on `TARGET_SPEND` or `MANUAL_CPC` once the campaign or its replacement scope has enough conversions. Switching is structural (DX-03, CHG-03)
DXP-07 (priority) | Priority interactions between overlapping Standard Shopping campaigns | Q1 `campaign.shopping_setting.campaign_priority` (0 low, 1 medium, 2 high), feed label, product scope (Q3c / Q6)                                                                                                                                                                                                                                                             | WARN per DXP-07 when a higher-priority campaign with a lower cap or bid overlaps a lower-priority one on the same products

**How priority works (public rule)**

-   Only matters when the same product is in several Standard Shopping campaigns
    for the same country (feed label).
-   The highest-priority campaign's bid is used, even if a lower-priority
    campaign bids more. If that campaign's budget runs out, the next priority
    can serve.
-   Equal priority: the higher bid is used.
-   So a **high-priority campaign with a \$0.05 cap** on popular products blocks
    a medium/low campaign with better bids from those auctions, and its own bid
    is too low to win them. Result: those products barely serve.

**Common pattern** Agent-built per-designer Max-clicks campaigns with
\$0.01–\$0.10 caps earned roughly half the value per \$1 of value-based Shopping
and PMax campaigns. Typical diagnosis: DXP-06 binding cap + BID-01 (click
strategy) + STR-01 (too little volume per campaign). The recommendation is
structural (consolidate into value-based campaigns, via experiment or staged
waves), so apply DX-03, CHG-03 and EXE-04, and point to
[account-structure-plan](../../account-structure-plan/SKILL.md).

**Gotchas**

-   A campaign may use a portfolio strategy (`campaign.bidding_strategy` set);
    read the cap from the `bidding_strategy` resource then.
-   There's no CHG rule for caps. Keep the budget as the spend guard, space cap
    changes like budget changes (CHG-01 spacing), and judge the result on value
    per \$1, not clicks.
-   An unset `cpc_bid_ceiling_micros` means no cap.
