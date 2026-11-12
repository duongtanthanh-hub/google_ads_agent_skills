# 1. Measurement (DX-04 step 1)

Check what's being counted before judging a Shopping campaign, even one on
Maximize clicks or Manual CPC: the evaluation (value per \$1, ROAS) still
depends on the conversion data.

Check           | What to verify                                                     | Data                                                                                                          | How to judge
--------------- | ------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------- | ------------
MEA-01          | Biddable goals are purchase-type                                   | Q3b (`campaign_conversion_goal.biddable`, `conversion_action.category`)                                       | BLOCK if no biddable purchase goal; WARN if a micro-conversion is biddable next to purchase. On click-based strategies the goal doesn't steer bids, but it still defines the "Conversions" column used to judge the campaign
MEA-02          | Purchase actions send real values                                  | Q3b `value_settings.always_use_default_value`                                                                 | WARN if default values are used on purchase; BLOCK if value bidding relies on an action without values
DXP-04          | Non-purchase share of counted conversions; value per conversion    | Q3 by `segments.conversion_action_category`, per window                                                       | Bands per DXP-04. Compare value per conversion with the account's purchase average for the same country and window
Settings change | Conversion settings changed in or before either window             | Bid-strategy status `LEARNING_CONVERSION_*` (Q1); ask the owner (conversion actions aren't in `change_event`) | WARN: comparisons across the change aren't like-for-like
Source match    | Ads value vs warehouse or GA4 revenue for the same orders and days | BI agent                                                                                                      | WARN if they diverge: tagging or attribution, not performance

**Patterns**

-   *Conversions up, value per conversion far below the account average*:
    non-purchase actions counted (common pattern: about \$0.85 per conversion on
    "All Products" campaigns vs \$15–21 elsewhere).
-   *All campaign types drop the same day*: site, tag or consent change.

**Gotchas**

-   `segments.conversion_action*` can't be selected with `metrics.cost_micros`;
    pull cost separately (Q2).
-   Enhanced conversions and consent mode aren't fully readable in GAQL; mark
    NOT_ASSESSED unless confirmed.
