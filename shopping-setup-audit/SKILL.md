---
name: shopping-setup-audit
description: >-
  Audits a Google Ads Standard Shopping campaign setup before launch, right
  after launch and on demand: measurement, bidding and CPC caps, budget,
  campaign priority and inventory filters, product-group structure, overlap
  with Performance Max on the same feed and country, negatives and client
  rules, launch batch size and change safety. Use when an agent plans, creates
  or edits Standard Shopping campaigns, ad groups, product groups or negatives,
  when a reviewer must approve such a change, or for a periodic Shopping
  configuration audit. Returns PASS/WARN/BLOCK findings with evidence. Don't
  use for Merchant Center feed and policy health (use feed-policy-audit), for
  Performance Max (use pmax-setup-audit), for why performance changed (use
  shopping-performance-diagnostic), or for Search or Demand Gen campaigns
  (use search-setup-audit or demandgen-setup-audit).
---

# Standard Shopping setup audit

Checks a Standard Shopping campaign (`advertising_channel_type = SHOPPING`)
against Google guidance and the advertiser's rules, then returns a verdict the
Ads Reviewer and the human approver can act on. Read-only: it never writes.

## When to use

-   **Before a write (`plan`):** the agent has a plan or payload to create or
    change Shopping campaigns, product groups, bids, budgets or negatives. Audit
    the plan, ideally with its `validate_only` result, before asking for
    approval.
-   **After launch (`post_launch`):** at D+1 confirm what's live matches the
    plan (products eligible, serving, no write failures), then D+3/7/14
    (EXE-08).
-   **On demand (`live`):** periodic audit of existing Shopping campaigns.

Not for: feed attributes, disapprovals and Merchant Center health
([feed-policy-audit](../feed-policy-audit/SKILL.md)); PMax setup
([pmax-setup-audit](../pmax-setup-audit/SKILL.md)); diagnosing a performance
change
([shopping-performance-diagnostic](../shopping-performance-diagnostic/SKILL.md));
incrementality tests
([test-design-and-power](../test-design-and-power/SKILL.md)); budget allocation
across campaigns
([budget-seasonality-plan](../budget-seasonality-plan/SKILL.md)); designing a
new structure ([account-structure-plan](../account-structure-plan/SKILL.md)).

## Inputs

Input                                                    | Required   | Notes
-------------------------------------------------------- | ---------- | -----
Mode                                                     | yes        | `plan`, `live` or `post_launch`
Customer ID and campaign IDs, or the plan/payload        | yes        | For `plan`: bid strategy, caps, budgets, priority, product-group tree, inventory filter, negatives
Target country, language, Merchant Center ID, feed label | yes        |
30-day baseline of comparable campaigns                  | yes        | Same country and product range: value-based Shopping and PMax cost, conversions, conversion value, average CPC; exclude brand traffic and the conversion-lag window (CHG-04)
Average order value (AOV)                                | for SHP-06 | From the warehouse or the purchase action; say which
`../_shared/client_rules.md`                             | yes        | Brand terms, protected products, intentional exclusions, approvers

## Pre-flight (standalone)

Same steps as the public `google-ads-api-account-diagnostics` skill (setup in
`google-ads-api-mcp-setup`, basics in `google-ads-api-quickstart`):

1.  Check the Google Ads MCP tools exist (`search`, `list_accessible_customers`,
    `get_resource_metadata`; names may carry a server prefix).
2.  No customer ID given: call `list_accessible_customers`, then run Q0 in
    [queries.md](references/queries.md) and use only enabled, non-manager client
    accounts.
3.  Tools missing: say so, give the GAQL from `queries.md` for a person to run,
    and mark the affected checks NOT_ASSESSED. Don't try scripts, local
    credentials or other ways around access.
4.  Before running new GAQL, check field names with `get_resource_metadata`.

## Workflow

1.  **Load the rules.** Read `../_shared/thresholds.md`,
    `../_shared/client_rules.md` and `../_shared/output_contract.md`. Use only
    thresholds from those files (SHP IDs included); never invent numbers.
2.  **Collect data.** `live`/`post_launch`: run the read-only queries in
    [queries.md](references/queries.md). `plan`: read values from the plan and
    fetch only the baseline, product and overlap data. Don't write anything.
3.  **Run all six checklists.** Mark checks without data NOT_ASSESSED.
    1.  [Measurement](references/01_measurement.md): MEA-01…06, SHP-06
    2.  [Bidding, CPC caps, budget](references/02_bidding_budget.md): SHP-01…02,
        BID-01…02, BUD-01, CHG-01…02
    3.  [Structure, priority, overlap](references/03_structure_priority.md):
        STR-01…04, SHP-03, SHP-04, SHP-07
    4.  [Products and feed (summary)](references/04_products_feed.md): FEED-01,
        FEED-02, FEED-04, FEED-05; deeper feed work goes to feed-policy-audit
    5.  [Negatives and client rules](references/05_negatives.md): STE-04, SHP-05
    6.  [Launch safety](references/06_launch_safety.md): EXE-01…08, CHG-03…05
4.  **Verdict and approval tier** per `output_contract.md` and EXE-05. In `live`
    mode with no change proposed, give the tier the recommended fix would need.
5.  **Reply** in the output-contract format: verdict line, ≤ 5-bullet summary,
    findings table, JSON block (`"skill": "shopping-setup-audit"`).
6.  **If BLOCK:** offer the smallest compliant alternative as numbered options
    with one recommended (e.g. one campaign per country with product groups by
    custom label instead of one campaign per seller; a pilot wave within STR-02;
    Maximize conversion value instead of capped Maximize clicks).

## Gotchas

-   **Clicks aren't the goal.** Maximize clicks and manual CPC optimise for
    clicks, not purchases. With tiny CPC caps they buy whatever traffic is left
    at that price, which usually returns little value per \$1. Judge Shopping by
    conversion value per \$1, not by CPC or click volume.
-   **Supported Standard Shopping bid strategies (API):** campaign-level
    `manual_cpc`, `target_spend` (Maximize clicks), `target_roas`,
    `maximize_conversion_value`; `maximize_conversions` only as a portfolio
    strategy.
-   **Priority only works between Standard Shopping campaigns.** It decides
    which Shopping campaign's bid is used for a product; with equal priority the
    higher bid wins. It has no effect on PMax.
-   **PMax vs Standard Shopping on the same products:** Ad Rank decides which
    one serves; PMax no longer wins automatically (changed in 2024, G). Don't
    assume a Shopping campaign is idle "because PMax wins".
-   **Feed label ≠ geo targeting.** `feed_label` selects products; location
    targeting decides where ads show. Check both.
-   **`EXCLUDED_PRODUCT_GROUPS` ("Products are excluded" in the UI)** isn't a
    `campaign.primary_status_reasons` value in API v24. Use the API checks in
    [04_products_feed.md](references/04_products_feed.md).
-   **Too early is an answer.** Don't recommend pausing, restructuring or
    switching strategy inside the CHG-03/DX-03 windows, except to fix serving or
    policy problems.
-   **A read-back isn't an audit,** and a person's request isn't approval to
    skip a BLOCK: return the finding, the risk and a staged option; execute the
    original only after explicit approval at the right tier (EXE-05).
-   **Money is in micros** (`cost_micros / 1,000,000`). Field names follow API
    v24; check them against your client library version.

## Sources

Public: Google Ads Help (Shopping bid strategies, campaign priority, product
groups, inventory filters, negative keywords, conversion goals and values, bid
strategy statuses, experiments), Merchant Center Help (feed labels, custom
labels, product data specification), Google Ads API developer guide ("Create a
Shopping campaign", "Create shopping listing groups", "Product reporting for
Shopping ads", mutate limits, batch jobs, quotas). Public skills referenced:
`google-ads-api-account-diagnostics`, `google-ads-api-mcp-setup`,
`google-ads-api-quickstart`. Labels **G** and **W** in `thresholds.md` mark
Google guidance still being confirmed and working proposals.
