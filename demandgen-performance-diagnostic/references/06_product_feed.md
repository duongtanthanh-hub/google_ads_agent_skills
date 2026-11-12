# 6. Product feed (Demand Gen with Merchant Center)

Run when a campaign has `campaign.shopping_setting.merchant_id` set or any
`DEMAND_GEN_PRODUCT_AD`. A product feed can also be attached to video and image
ads unless `campaign.shopping_setting.disable_product_feed` is true.

Check            | What to verify                                                                                               | Data                                                                                                                                                                                                                                                                   | How to judge
---------------- | ------------------------------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
FEED-01          | Right Merchant Center account and a feed label with eligible products for the campaign's market and language | Q1b: `campaign.shopping_setting.merchant_id`, `.feed_label`, `.disable_product_feed`; Q15: `shopping_product` in campaign scope                                                                                                                                        | BLOCK if no eligible products for that feed label (product ads can't serve)
FEED-02          | Share of products in scope that are `NOT_ELIGIBLE`                                                           | Q15: `shopping_product.status` (`ELIGIBLE`, `ELIGIBLE_LIMITED`, `NOT_ELIGIBLE`), `shopping_product.issues` (`error_code`, `ads_severity`, `affected_regions`)                                                                                                          | Per FEED-02 bands. List the top 5 `error_code` values with counts. A jump in `NOT_ELIGIBLE` that matches the drop date is a Feed cause (measured if the counts are stored from before; otherwise inferred)
FEED-04          | Products excluded on purpose vs by accident                                                                  | Products in Merchant Center for the feed label (account scope) vs products returned in campaign scope (Q15); the product filter set up for the campaign (UI; check where your API version exposes it); `client_rules.md` intentional exclusions and protected products | WARN for unlisted exclusions; BLOCK if a protected product is excluded
Product delivery | Which products get impressions and clicks                                                                    | Q15 metrics: `metrics.impressions`, `metrics.clicks`, `metrics.ctr`                                                                                                                                                                                                    | In Demand Gen scope only these three metrics are available, and date segmentation isn't allowed (date filters are). Use them to spot products that never serve, not to compute ROAS
Feed vs creative | Product ads vs other ad types                                                                                | Q11c: results by `ad_group_ad.ad.type`                                                                                                                                                                                                                                 | If only product-ad conversions fell, check the feed first; if all types fell, look elsewhere

**Notes**

-   `shopping_product.issues` can lag behind a feed change; re-check the next
    day before calling a fix failed.
-   Large print-on-demand catalogues: one disapproval reason (e.g. image or
    trademark policy) can remove thousands of near-identical items at once. Fix
    in Merchant Center; see `../feed-policy-audit/SKILL.md` for the full feed
    review.
-   Product ads need the feed label's country and language to match where the ad
    group serves (with upgraded targeting, location and language can be set per
    ad group; check `campaign.demand_gen_campaign_settings.upgraded_targeting`).
