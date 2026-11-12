# 1. Goal, bidding and budget

Demand Gen supports Maximize conversions (optional target CPA), Maximize
conversion value (optional target ROAS), and click-based strategies (Maximize
clicks, target CPC). For a sales goal, bid on conversions.

Check  | What to verify                                                                     | Data                                                                                                               | How to judge
------ | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ | ------------
DG-01  | Bidding matches the goal                                                           | `campaign.bidding_strategy_type`; stated goal                                                                      | PASS: `MAXIMIZE_CONVERSIONS` / `MAXIMIZE_CONVERSION_VALUE` (or `TARGET_CPA` / `TARGET_ROAS`) on a sales goal. WARN: `TARGET_SPEND` (Maximize clicks) or `TARGET_CPC` on a sales goal, or with no documented traffic objective
BID-01 | Value-based bidding once purchase values are tracked                               | `bidding_strategy_type`, `conversion_action.value_settings`                                                        | WARN if `MAXIMIZE_CONVERSIONS` is used while values are tracked **and** MEA-06 volume is met. Below MEA-06, Maximize conversions without a target is a sensible start; say so
BID-02 | Starting target anchored to history                                                | Planned tCPA/tROAS vs 30-day actual of comparable non-brand campaigns (same countries)                             | PASS within ±20%; WARN 20–50% stricter; BLOCK > 50% stricter. For a first Demand Gen campaign with no history, prefer no target (MEA-06)
MEA-06 | Enough conversions to use a target                                                 | 30-day primary conversions of the campaign or the campaigns it replaces                                            | PASS ≥ 30; WARN 15–29; below 15 launch without a target and say so
BUD-01 | Basic budget headroom                                                              | `campaign_budget.amount_micros` / 1e6 vs target CPA (or 30-day cost per conversion)                                | WARN if < 3×
DG-02  | Demand Gen budget sizing                                                           | Same ratio as BUD-01                                                                                               | PASS ≥ 15×; WARN < 15×. Report BUD-01 too when < 3×
DG-03  | Evaluation readiness (live mode, or when someone asks for a performance judgement) | Conversions since launch or since the last bid-strategy/target change (`change_event`, Q9); days since that change | PASS ≥ 50 conversions **and** CHG-03/CHG-04 met. Otherwise WARN "too early": report the trend, make no structural recommendation
CHG-01 | Budget change step (edits)                                                         | Old vs new budget; last budget change date (`change_event`, Q9)                                                    | PASS: ≤ 20% and ≥ 3 days since the last budget change. WARN: 20–50%, or < 3 days since the last change. BLOCK: > 50% in one step, unless the EXE-05 approval names the risk
CHG-02 | Target change step (edits)                                                         | Old vs new target; last target change date (Q9)                                                                    | PASS: ≤ 15% and ≥ 7 days since the last target change. WARN: 15–30%, or < 7 days. BLOCK: > 30% in one step without EXE-05 approval

**Ad group overrides.** `ad_group.target_cpa_micros` / `ad_group.target_roas`
override the campaign target. Check `ad_group.effective_target_cpa_micros` and
`ad_group.effective_target_roas` so BID-02 uses the target actually in force.

**Calculations**

-   Budget ratio = daily budget / target CPA. With value bidding and no tCPA,
    use the 30-day cost per conversion of the comparable campaigns.
-   If DG-02 fails but budget can't grow, suggest fewer ad groups (so each gets
    more budget) or one campaign instead of several, rather than a stricter
    target.

**Common pattern**

-   Demand Gen is a small share of spend. Spreading it over many small campaigns
    will fail DG-02 and DG-03 everywhere and produce no usable result. Prefer
    one or two well-funded campaigns per market cluster.
-   "Maximize clicks with a low CPC cap" buys cheap, low-intent traffic. It
    delivered much lower value per \$1 than value-based bidding in the
    advertiser's own Shopping data; don't copy it to Demand Gen.
