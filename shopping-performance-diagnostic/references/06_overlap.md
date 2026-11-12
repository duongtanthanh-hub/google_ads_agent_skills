# 6. Overlap with Performance Max and other Shopping campaigns

When the same products are in several campaigns, one campaign's drop is often
another's gain. Check the whole set before calling it a loss.

Check             | What to verify                                                  | Data                                                                                                                                                                             | How to judge
----------------- | --------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
STR-03            | Same products targeted by several campaigns in the same country | Q6 across `advertising_channel_type IN ('SHOPPING', 'PERFORMANCE_MAX')` grouped by `product_item_id` and `product_feed_label`; product-group scope (Q3c) and PMax listing groups | WARN if overlap isn't documented in `client_rules.md`
DXP-07 (shift)    | Spend moved between overlapping campaigns                       | Q6 per window: cost per campaign on the shared items                                                                                                                             | Bands per DXP-07. If the account total on the shared items is flat while this campaign fell, report "shifted, not lost"
DXP-07 (priority) | Priority conflicts between Standard Shopping campaigns          | `campaign.shopping_setting.campaign_priority` and caps/bids (step 3)                                                                                                             | WARN when a higher-priority campaign with a lower cap overlaps a better-bidding lower-priority one
Account view      | Did the account lose value, or just this campaign?              | Q2 for all Shopping + PMax campaigns on the same feed label                                                                                                                      | Diagnose at account level when the account total is flat

**Rules (public and guidance)**

-   Between Standard Shopping campaigns: priority first, then bid (P).
-   Between PMax and Standard Shopping on the same products: Ad Rank decides
    which one serves; PMax no longer gets automatic priority (G, public wording
    to confirm). So new or re-bid Shopping campaigns can take traffic from PMax,
    and the other way round.
-   Overlap isn't always wrong (e.g. a deliberate test), but it must be written
    down (STR-03) and read as one system.

**Common pattern** Hundreds of per-designer Shopping campaigns can cover the
same items as category PMax. A PMax drop the week after a launch wave may be
traffic shifting into low-cap Shopping campaigns with lower value per \$1, not a
PMax problem. Recommend fixing the overlap (exclusive product scopes via custom
labels) through [account-structure-plan](../../account-structure-plan/SKILL.md),
with DX-03 and EXE-04.

**Gotchas**

-   Join on item ID **and** feed label (country). The same item in different
    countries isn't overlap.
-   PMax product rows also come from `shopping_performance_view`; filter by
    `campaign.advertising_channel_type` to separate them.
-   A shift labelled "measured" needs product-level cost moving between the
    campaigns in the same window; otherwise it's inferred.
