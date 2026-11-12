# 5. Products and eligibility (DX-04 steps 2 and 4)

Which products moved, and did they lose eligibility, scope, or competitiveness?

Check             | What to verify                               | Data                                                                                                                                                                                     | How to judge
----------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
DXP-01 (products) | Products or segments that explain the change | Q6 `shopping_performance_view` per window by `segments.product_item_id`, then `product_type_l1` and `product_custom_attribute0…4`                                                        | Name the movers covering the DXP-01 share of the absolute change in value or cost, with cost, value and value per \$1 in both windows
Product groups    | Product-group performance and bids           | Q5 `product_group_view` metrics joined to the Q3c tree (`ad_group_criterion.listing_group.*`, `negative`, `effective_cpc_bid_micros`)                                                    | A product group that lost most impressions after a bid cut or exclusion = measured cause (step 4)
FEED-02           | Eligibility of products in scope now         | Q7 `shopping_product.status`, `issues` (campaign filter)                                                                                                                                 | Bands per FEED-02; list the top 5 issue types. Lost items that are now `NOT_ELIGIBLE` = measured eligibility cause
FEED-04           | Excluded product groups                      | Q3c rows with `ad_group_criterion.negative = true`                                                                                                                                       | WARN if not listed in `client_rules.md`; BLOCK if a protected product is excluded
Benchmark CTR     | Click-through vs similar products            | `metrics.benchmark_ctr` vs `metrics.ctr` (Q3d; check selectable resource first)                                                                                                          | Context only: much lower CTR than benchmark suggests title, image or price issues (inferred)
Price benchmarks  | Your price vs market benchmark               | Merchant API reports: `price_competitiveness_product_view` (`price`, `benchmark_price`); `price_insights_product_view` (`suggested_price`, predicted impression/click/conversion change) | Only with Merchant API read access and eligibility; otherwise NOT_ASSESSED. A price moved above benchmark on top movers = inferred cause; confirm with price history

**Reading product moves**

-   *Fewer items serving*: eligibility, feed label or exclusions (measured if Q7
    or Q3c confirm).
-   *Same items, fewer clicks*: cap or bid (step 3), overlap (step 6), or
    competitiveness (benchmarks).
-   *Same clicks, lower value per \$1*: measurement (step 1), price or
    landing-page change (inferred), or seasonal fade (step 7).

**Gotchas**

-   `shopping_performance_view` counts product impressions, not ad impressions;
    compare cost, conversions and value across reports.
-   Product dimensions are recorded as of the event; renamed product types show
    as two rows.
-   `shopping_product` is a current snapshot; it can't prove when an item became
    ineligible, and `issues` can lag up to 24 hours.
-   Benchmark metrics and price data describe the market, not causes. Label any
    conclusion from them *inferred*.
-   The Merchant API is a separate API with its own access. Don't request new
    access or credentials as part of this skill; report NOT_ASSESSED instead.
