# 5. Audiences

In Demand Gen, audiences are signals and targeting inputs for an ad group. Good
first-party inputs help the system find buyers faster; optimized targeting and
lookalike expansion then look beyond them.

Check | What to verify                                      | Data                                                                                                                                                                                                                                                                                                            | How to judge
----- | --------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
DG-09 | First-party audience signal and lookalike seed size | `ad_group_criterion` with `type IN ('USER_LIST', 'AUDIENCE', 'CUSTOM_AUDIENCE')`; `user_list.type`, `user_list.size_range_for_display`, `user_list.size_for_display`, `user_list.eligible_for_display`; for lookalikes `user_list.lookalike_user_list.seed_user_list_ids`, `.expansion_level`, `.country_codes` | PASS: each ad group has ≥ 1 audience built on first-party data (customer list, site visitors or purchasers, or a lookalike seeded from them), and every lookalike seed is ≥ 1,000 matched users. WARN: no first-party-based audience, or a seed below 1,000
DG-10 | Optimized targeting                                 | `ad_group.optimized_targeting_enabled`; `ad_group.exclude_demographic_expansion`                                                                                                                                                                                                                                | PASS: on with conversion-based bidding, or off with a recorded reason. WARN: off with no reason

**Reading seed size**

-   Seed lists are the `user_list` IDs in
    `lookalike_user_list.seed_user_list_ids`. Query those lists and read
    `size_range_for_display`. `LESS_THAN_FIVE_HUNDRED` and
    `LESS_THAN_ONE_THOUSAND` fail the DG-09 seed size check.
-   Google sets its own minimum seed size for creating a lookalike. A seed below
    1,000 may still work, but the model has little to learn from.
-   `lookalike_user_list.country_codes` should cover the campaign's markets; a
    US-seeded lookalike restricted to `US` won't help a German ad group.

**Expansion level** (`expansion_level`: `NARROW`, `BALANCED`, `BROAD`)

-   Narrow = closest to the seed, smallest reach; Broad = largest reach.
    Balanced is a reasonable default. Note that Google may use lookalikes as a
    suggestion rather than a hard limit; judge results on conversions.

**Good first-party inputs for print-on-demand**

-   Past purchasers (Customer Match), split by recency if large enough.
-   Site visitors who viewed or added to cart in the last 30 days.
-   Lookalike of purchasers, per market.
-   Custom segments built from top-converting search terms (e.g. "personalised
    name necklace", "custom pet portrait").

**Exclusions.** If the goal is new customers, consider excluding recent
purchasers, or use the new-customer goal where available. Record which choice
was made so DG-11 compares like with like.

**Privacy.** Customer lists must follow Customer Match policy and the market's
consent rules (MEA-04 for EEA/UK). Never paste customer data into the audit
output.
