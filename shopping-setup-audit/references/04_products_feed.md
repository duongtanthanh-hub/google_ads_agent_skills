# 4. Products and feed (summary for Shopping)

Quick checks that the campaign has products it can show. For attribute quality,
disapproval types, custom-label hygiene and Merchant Center account health, run
[feed-policy-audit](../../feed-policy-audit/SKILL.md).

Check   | What to verify                                              | Data                                                                                                                                                                                                                                   | How to judge
------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
FEED-01 | Right Merchant Center and feed label for the target country | `campaign.shopping_setting.merchant_id`, `feed_label`; linked accounts (`product_link`, Q0b); count of products at campaign scope (Q6)                                                                                                 | BLOCK if the campaign includes no eligible products (e.g. a `DE` campaign pointing at a feed label that only has UK items). If `feed_label` is empty, the campaign uses all feeds in the Merchant Center: check that products for the target country are among them
FEED-02 | Product eligibility in scope                                | `shopping_product.status` at campaign scope (Q6): `ELIGIBLE`, `ELIGIBLE_LIMITED`, `NOT_ELIGIBLE`                                                                                                                                       | Share `NOT_ELIGIBLE`: < 5% PASS, 5–20% WARN, > 20% BLOCK. List the top 5 `issues.error_code` values with counts
FEED-04 | Product exclusions are intended                             | Negative listing groups (`ad_group_criterion.negative = TRUE` with `listing_group`, Q4), inventory filter (Q5), the "Everything else" node; recommendations `SHOPPING_TARGET_ALL_OFFERS` and `SHOPPING_ADD_PRODUCTS_TO_CAMPAIGN` (Q12) | PASS if every exclusion is listed in `client_rules.md`; WARN if not; BLOCK if a protected product is excluded
FEED-05 | Custom labels carry business priorities                     | Custom labels used in product groups or the inventory filter                                                                                                                                                                           | WARN if margin, bestseller, season or designer priorities exist but aren't encoded as custom labels

**`EXCLUDED_PRODUCT_GROUPS` ("Products are excluded")**

-   The Google Ads UI can show this campaign status when some products are
    excluded. It is **not** a `CampaignPrimaryStatusReason` value in API v24, so
    don't wait for it in `campaign.primary_status_reasons`.
-   It means "some products are excluded", not "nothing serves". Campaigns
    showing it often spend normally.
-   API checks instead: (1) negative unit nodes in the product-group tree and
    what they exclude; (2) the inventory filter; (3) the recommendations above,
    including `shopping_add_products_to_campaign_recommendation.reason`
    (`ALL_PRODUCTS_ARE_EXCLUDED_FROM_CAMPAIGN`,
    `ADS_ACCOUNT_EXCLUDES_OFFERS_FROM_CAMPAIGN`) and
    `shopping_target_all_offers_recommendation.untargeted_offers_count`; (4)
    compare products at campaign scope with products at account scope for the
    same feed label (Q6 vs Q6c).
-   Then judge with FEED-04. For a per-seller campaign, "everything except this
    seller is excluded" is expected by design; the structural finding is
    STR-01/SHP-04, not FEED-04.

**Product-group tree sanity (from Q4)**

-   At least one positive unit node, otherwise ads won't serve.
-   Every subdivision has an "Everything else" child (the API rejects trees
    without it). Check whether it is bid on or excluded, and whether that
    matches the plan.
-   Manual CPC: positive unit nodes need a bid; compare bids with SHP-02.
-   `shopping_product.effective_max_cpc_micros` (campaign or ad group scope,
    manual bidding only) shows the highest bid that applies to each product.
