# 4. Product feed (product ads)

A Merchant Center feed lets Demand Gen show products from the catalogue (product
ads, and products alongside image and video ads).

Check   | What to verify                                                       | Data                                                                                                                                                                       | How to judge
------- | -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
DG-08   | Feed attached when selling products                                  | `campaign.shopping_setting.merchant_id`, `campaign.shopping_setting.feed_label`, `campaign.shopping_setting.disable_product_feed`; presence of `DEMAND_GEN_PRODUCT_AD` ads | PASS: a feed is attached for the campaign's market, or a reason for not using it is recorded. WARN: sales goal, eligible products exist for the market, and no feed is attached with no reason
FEED-01 | Feed label has eligible products for the target country and language | `shopping_product` rows for the Merchant Center ID and feed label (Q7)                                                                                                     | BLOCK if there are no eligible products for that feed label
FEED-02 | Product eligibility                                                  | `shopping_product.status`, `shopping_product.issues` for products in scope                                                                                                 | `NOT_ELIGIBLE` share: < 5% PASS; 5–20% WARN; > 20% BLOCK. List the top 5 issue types

For a full feed and policy review, use
[feed-policy-audit](../../feed-policy-audit/SKILL.md).

**Notes**

-   `disable_product_feed = true` turns the optional feed off. That's fine for
    pure brand or video-story campaigns; record why.
-   The feed label must match the market. A campaign for one country pointed at
    a feed label that only holds another country's items will create
    successfully but show no products.
-   Product filters (which items a product ad group shows) are set in the UI or
    API per ad group. If the filter can't be read, mark the filter part of DG-08
    NOT_ASSESSED and ask for the plan's filter definition.

**Common pattern**

-   With a large print-on-demand catalogue, filter product ad groups by custom
    label (season, bestseller, margin tier) or product type, not by seller or
    individual item.
-   Check that protected products in `client_rules.md` are not filtered out.
