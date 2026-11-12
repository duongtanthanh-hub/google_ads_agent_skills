---
name: shopping-performance-diagnostic
description: >-
  Diagnoses why a Google Ads Standard Shopping campaign's results changed:
  conversion value or ROAS drops, underspend or lost traffic, CPC or bid-cap
  limits, campaign-priority conflicts, product eligibility changes and overlap
  with Performance Max. Works through a fixed root-cause order (measurement,
  delivery and impression share, bids and priority, recent changes, products,
  overlap, conversion lag and seasonality) using read-only GAQL, and labels
  every cause as measured or inferred. Use when someone asks "why did
  Shopping drop / stop spending / get so few clicks?", when a post-change
  check finds a material change, or before proposing Shopping bid, cap,
  priority or budget changes. Don't use for setup checks before launch (use
  shopping-setup-audit), reviewing a proposed write (use
  ads-change-guardrails), Performance Max (use pmax-performance-diagnostic),
  feed-policy work (use feed-policy-audit) or incrementality questions (use
  lift-and-geo-experiments or causal-impact-analysis).
---

# Standard Shopping performance diagnostic

Finds the most likely cause of a Standard Shopping performance change, with
evidence, in a fixed order. Read-only: it never writes to Google Ads.

## When to use

-   **"Why did it change?"** Value, ROAS, clicks or spend moved between two
    periods on one or more Standard Shopping campaigns.
-   **Few clicks or underspend**, especially on Maximize clicks or Manual CPC
    campaigns with low CPC caps.
-   **Post-change follow-up.** A D+3/D+7/D+14 check (EXE-08,
    [ads-post-change-check](../ads-post-change-check/SKILL.md)) found a material
    change after a bid, cap, priority, budget or product-group edit.
-   **Before a recommendation** on caps, bids, priority, budget or moving
    products to or from Performance Max.

Not for: setup checks
([shopping-setup-audit](../shopping-setup-audit/SKILL.md)), feed and policy
problems ([feed-policy-audit](../feed-policy-audit/SKILL.md)), Performance Max
([pmax-performance-diagnostic](../pmax-performance-diagnostic/SKILL.md)), or
incrementality
([lift-and-geo-experiments](../lift-and-geo-experiments/SKILL.md)).

## Inputs

Input                                     | Required                         | Notes
----------------------------------------- | -------------------------------- | -----
Customer ID and campaign IDs              | yes (or find them in pre-flight) | Client accounts only, never manager accounts
Symptom and dates                         | yes                              | If no dates, compare the last 14 complete days with the 14 before (DX-01)
Bid strategy goal and KPI source of truth | yes                              | e.g. "value per \$1 ≥ 1.0 on warehouse revenue"
`../_shared/client_rules.md`              | yes                              | Brand terms, protected products, intentional exclusions, documented overlaps
Merchant API read access                  | optional                         | Only for price benchmarks (step 5); otherwise NOT_ASSESSED

## Pre-flight (standalone version of the official `google-ads-api-account-diagnostics` skill)

1.  **Tools.** Check that the Google Ads MCP read tools are available (`search`,
    `list_accessible_customers`, `get_resource_metadata`; they may have a server
    prefix). If not, go to step 4.
2.  **Account.** If no customer ID was given, call `list_accessible_customers`,
    then run [Q0](references/queries.md) and keep only `ENABLED`, non-manager
    client accounts. Note the currency and time zone.
3.  **Fields.** Before running any query that isn't in `queries.md`, check the
    field names and which resources a metric can be selected with, using
    `get_resource_metadata` (EXE-07). Queries here use API v24 names.
4.  **Fallback (no MCP tools).** Say that live diagnosis needs the Google Ads
    MCP server (setup: the `google-ads-api-mcp-setup` skill), then hand over the
    queries, windows and decision rules for a manual run. Don't try other ways
    to reach account data.

## Workflow

1.  **Load the rules.** Read `../_shared/thresholds.md`,
    `../_shared/client_rules.md` and `../_shared/output_contract.md`. Use only
    thresholds from those files (DX, DXP, MEA, CHG, BUD, BID, FEED, STR); never
    invent numbers.
2.  **Frame the question.** Set the windows per DX-01 (lag excluded per DXP-05
    and CHG-04), flag events, and size the change with DX-02. Below the floor:
    report "noise / too little data" after steps 3.1–3.3 anyway.
3.  **Walk the root-cause order (DX-04).** Record evidence for every step. Steps
    1–4 always run; stop going deeper once the causes found explain the change
    (DXP-01).
    1.  [Measurement](references/01_measurement.md): MEA-01/02, DXP-04
    2.  [Delivery and impression share](references/02_delivery_impression_share.md):
        status, budget (DXP-02, BUD-01), impression share (DXP-03)
    3.  [Bids, CPC caps and priority](references/03_bids_caps_priority.md):
        DXP-06, DXP-07, bid-strategy status, BID-01
    4.  [Recent changes](references/04_recent_changes.md): CHG-01…03, DX-03
    5.  [Products and eligibility](references/05_products.md): DXP-01,
        FEED-02/04, benchmark signals
    6.  [Overlap with PMax and other Shopping campaigns](references/06_overlap.md):
        STR-03, DXP-07
    7.  [Conversion lag and seasonality](references/07_lag_seasonality.md):
        DXP-05, DXP-08, DX-01
4.  **Label every cause (DX-05)** as *measured* or *inferred*, with its window.
5.  **Decide the verdict and tier** (below) and reply in the output-contract
    format, adding `windows` and `root_causes` keys to the JSON block.
6.  **Recommend.** Numbered options with one recommended. Change options must
    pass CHG-01…03 and state their EXE-05 tier. Structural options (bid-strategy
    switch, consolidation, moving products to PMax) also need DX-03.

## Verdict for a diagnostic

-   **BLOCK**: a finding makes performance-driven changes unsafe until fixed: a
    measurement BLOCK (MEA-01, MEA-02, DXP-04), a campaign that can't serve
    (`primary_status` `NOT_ELIGIBLE` or `MISCONFIGURED`, FEED-01 BLOCK, FEED-02
    BLOCK), or a requested action that breaks a CHG BLOCK band.
-   **WARN**: a material change (DX-02) with a diagnosed cause, a "too early"
    result (DX-03), or any other WARN finding. A rule already broken in live
    settings (BID-02 or STR-01 BLOCK band, a past CHG step) is a WARN cause
    here; its BLOCK band applies to new proposals.
-   **PASS**: no material change and no WARN findings.
-   **Approval tier**: the EXE-05 tier of the recommended action; T1 if the
    recommendation is only to monitor.

## Gotchas

-   **Maximize clicks optimises clicks, not sales.** Low value per \$1 on a
    Max-clicks campaign with a tiny CPC cap is expected behaviour, not a
    mystery. The question is whether the campaign should exist in that form
    (BID-01, STR-01), which is a structural change (DX-03).
-   **A CPC cap that's far below market CPC** stops a campaign from winning
    auctions: low impression share lost to rank, `LIMITED_BY_CPC_BID_CEILING`,
    underspend (DXP-06). Raising the budget does nothing.
-   **Priority beats bids** between Standard Shopping campaigns sharing products
    (DXP-07): a high-priority campaign with a tiny cap can take traffic from a
    better campaign.
-   **PMax vs Standard Shopping** on the same products is decided by Ad Rank,
    not automatic PMax priority (G). Spend can swing between them.
-   **Impression share for Shopping** reflects Google Search inventory. Use
    search impression share, budget-lost and rank-lost share, absolute-top share
    and click share (documented for Shopping); skip exact-match share, which is
    keyword-based.
-   **`EXCLUDED_PRODUCT_GROUPS` isn't a v24 API enum value.** Read excluded
    product groups from `ad_group_criterion.negative` on listing groups.
-   **Price benchmarks aren't in the Google Ads API.** They come from Merchant
    Center (Merchant API reports). Don't infer price competitiveness from CTR.
-   **Small campaigns are noise** (DX-02); diagnose them at portfolio level.

## Sources

Public: Google Ads Help (impression share, click share, campaign priority for
Shopping campaigns, bid strategy statuses, average daily budgets, seasonality
adjustments, data exclusions), Merchant Center Help (price competitiveness),
Google Ads API docs (`shopping_performance_view`, `product_group_view`,
`change_event`, `shopping_product`), Merchant API reports, and the official
`google-ads-api-account-diagnostics` skill. The PMax/Standard Shopping Ad Rank
rule is **G** (public wording to confirm).
