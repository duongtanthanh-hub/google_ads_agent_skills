# 3. Structure, campaign priority and overlap

Many small campaigns split the data so that none gets enough conversions to
learn. Prefer fewer campaigns, with product groups (or inventory filters) to
segment, when the goal and budget are the same.

Check  | What to verify                                                      | Data                                                                                                                                                            | How to judge
------ | ------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
STR-01 | Each new campaign can reach enough volume                           | Historical 30-day conversions for the products and country the campaign will cover (from `shopping_performance_view` by item, product type or custom label, Q7) | PASS ≥ 30 projected; WARN 15–29; BLOCK < 15. Then suggest a product group inside an existing campaign with the same goal
STR-02 | Launch batch size                                                   | New campaigns in this plan + created in the last 7 days (`change_event` CREATE on CAMPAIGN, Q10); active campaigns in the account                               | PASS ≤ max(5, 10% of active); else WARN if a pilot wave passed (EXE-04), BLOCK if not
STR-03 | Product overlap with other campaigns in the same country            | Item IDs included by this campaign vs enabled PMax and Shopping campaigns on the same Merchant Center ID, feed label and country (Q6, Q6b)                      | WARN if the same products are targeted twice and `client_rules.md` doesn't say it's intended
STR-04 | Labels and naming for agent-created entities                        | `campaign.name`, `campaign.labels` + `label.name` (Q11)                                                                                                         | WARN if the agreed label or naming is missing
SHP-03 | Campaign priority is deliberate where Shopping campaigns overlap    | `campaign.shopping_setting.campaign_priority` (0 = Low, 1 = Medium, 2 = High) of overlapping Standard Shopping campaigns; their budgets, caps and negatives     | PASS: no Shopping-to-Shopping overlap, or priorities differ by design and the plan says which campaign should win which queries. WARN: overlap with equal priorities, or a higher-priority campaign with a smaller budget or tighter cap in front of the better-performing one
SHP-04 | Segmentation uses durable product attributes                        | Inventory filter (`campaign_criterion.listing_scope`, Q5) and product-group tree (`ad_group_criterion.listing_group`, Q4)                                       | PASS: split by custom label, product type or category. WARN: split by long item-ID lists, or one campaign per seller/designer/brand value (if STR-01 fails as well, STR-01 carries the BLOCK)
SHP-07 | Live campaigns on click-based bidding earn their keep (`live` only) | 30-day mature value per \$1 vs the value-based Shopping/PMax baseline in the same country (DX-01 windows, DX-02 floor)                                          | PASS at or above the SHP-07 ratio. WARN below it: suggest consolidating into the value-based campaign and hand the detail to [shopping-performance-diagnostic](../../shopping-performance-diagnostic/SKILL.md). Below the DX-02 volume floor: report "too little data"

**Campaign priority, in short (P: About campaign priority)**

-   Only matters when the same product could serve from more than one **Standard
    Shopping** campaign in the same country.
-   The highest-priority campaign's bid is used, even if a lower-priority
    campaign bids more. If it runs out of budget, the next priority level can
    serve. Equal priority → the higher bid is used.
-   Common deliberate use: a high-priority "catch-all" campaign with low bids
    and negatives for generic queries, and a lower-priority campaign with higher
    bids for specific queries. If nobody can explain the design, flag SHP-03.

**Overlap with Performance Max (STR-03)**

-   PMax and Standard Shopping on the same products, Merchant Center, feed label
    and country compete in the same auctions; Ad Rank decides which serves (G).
    Overlap splits conversion data between two strategies and makes both slower
    to learn.
-   Intended overlap (an experiment arm, a search-term control campaign) must be
    written in `client_rules.md` or the plan. Otherwise suggest excluding the
    products from one side (listing group in PMax or product group in Shopping)
    or folding the Shopping products into the PMax asset group.

**When a separate Shopping campaign is justified** (otherwise use product groups
in an existing campaign)

-   A different goal or target (e.g. a margin tier needing its own tROAS).
-   A ring-fenced budget (seasonal, new market).
-   A different country, language or feed label.
-   A planned experiment arm.
-   Search-term control the business needs (negatives by query intent).

**The advertiser note:** one campaign per designer or seller (hundreds of
campaigns, about 2 projected conversions a month each) fails STR-01 and STR-02.
Put the designer or seller in a custom label and use it in product groups inside
one campaign per country and goal.
