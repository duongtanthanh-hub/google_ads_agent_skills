---
name: demandgen-setup-audit
description: >-
  Audits a Google Ads Demand Gen campaign setup before launch and on demand:
  goal and bidding, budget sizing against target CPA, channel controls, image,
  video and text coverage (including vertical 9:16 for YouTube Shorts), product
  feed, audiences and lookalike seeds, optimized targeting, conversion
  definition (click, engaged-view, view-through), the incrementality
  measurement plan, and launch safety. Use when an agent plans, creates or
  edits a Demand Gen campaign or ad group, when a reviewer must approve such a
  change, or for a periodic Demand Gen configuration audit. Returns
  PASS/WARN/BLOCK findings with evidence. Don't use for diagnosing performance
  changes (use demandgen-performance-diagnostic), for running or reading a
  lift study (use lift-and-geo-experiments / test-design-and-power), or for
  PMax, Shopping or Search campaigns (use pmax-setup-audit,
  shopping-setup-audit, search-setup-audit).
---

# Demand Gen setup audit

Checks a Demand Gen campaign against Google best practice and the advertiser's
own rules, then returns a verdict the Ads Reviewer and the human approver can
act on. Read-only: it never writes to Google Ads.

## When to use

-   **Before a write:** the agent has a plan or payload to create or change a
    Demand Gen campaign, ad group or ad. Audit the **plan**, ideally with its
    `validate_only` result, before asking for approval.
-   **After launch:** at D+1 confirm that what's live matches the plan (ads
    approved, channels and audiences as planned, serving).
-   **On demand:** periodic configuration audit of existing Demand Gen
    campaigns.
-   **Before a budget decision about Demand Gen:** check that the measurement
    plan (DG-11, DG-12) can actually answer "does Demand Gen work?".

Not for: why performance moved (diagnostic skill), running or reading a lift
study (media-effectiveness skills), or splitting budget across channels
(planning skills).

## Inputs

Input                                                          | Required             | Notes
-------------------------------------------------------------- | -------------------- | -----
Mode                                                           | yes                  | `plan` (proposed config or payload), `live` (existing campaign IDs), or `post_launch`
Customer ID and campaign/ad group IDs, or the plan             | yes                  |
Campaign goal                                                  | yes                  | Sales (purchase), new customers, or traffic; drives DG-01 and DG-11
Target countries/languages; Merchant Center ID for product ads | yes / if product ads |
30-day baseline of comparable campaigns                        | yes                  | CPA, ROAS, conversions; exclude brand Search
Measurement plan                                               | for budget decisions | Lift study or geo experiment design, if any
`../_shared/client_rules.md`                                   | yes                  | Brand terms, protected products, approvers

## Workflow

1.  **Load the rules.** Read `../_shared/thresholds.md`,
    `../_shared/client_rules.md` and `../_shared/output_contract.md`. Use only
    thresholds from those files; never invent numbers.
2.  **Pre-flight (read access).** Short version of the public
    `google-ads-api-account-diagnostics` pre-flight:
    -   Check that the Google Ads MCP tools `search`,
        `list_accessible_customers` and `get_resource_metadata` are available
        (they may carry a prefix).
    -   If no customer ID is given, call `list_accessible_customers`, then query
        `customer_client` for enabled, non-manager accounts (Q0 in
        [queries.md](references/queries.md)). Never query manager or closed
        accounts.
    -   Check fields you haven't used before with `get_resource_metadata`
        (EXE-07).
    -   **If the tools are missing:** say so, point to the public
        `google-ads-api-mcp-setup` skill, and return the GAQL in
        [queries.md](references/queries.md) for a person to run. Don't probe
        local services, scripts or credentials.
3.  **Collect data.** For `live` and `post_launch`, run the read-only queries in
    [queries.md](references/queries.md). For `plan`, read values from the plan
    and fetch only the baseline, audience and feed data. Don't write anything.
4.  **Run all seven checklists.** Don't skip any; checks without data are
    NOT_ASSESSED.
    1.  [Goal, bidding and budget](references/01_goal_bidding_budget.md):
        DG-01…03, BID-01…02, BUD-01, MEA-06, CHG-01…02
    2.  [Channels and targeting](references/02_channels_targeting.md): DG-04,
        SRCH-13
    3.  [Creative](references/03_creative.md): DG-05…07, AST-08
    4.  [Product feed](references/04_product_feed.md): DG-08, FEED-01…02
    5.  [Audiences](references/05_audiences.md): DG-09…10
    6.  [Conversions and measurement](references/06_measurement.md): MEA-01…05,
        DG-11…12, CHG-05
    7.  [Launch safety](references/07_launch_safety.md): STR-01…04, EXE-01…08,
        CHG-03…04
5.  **Decide the verdict and approval tier** per `output_contract.md` and
    EXE-05.
6.  **Reply** in the output-contract format
    (`"skill": "demandgen-setup-audit"`): verdict line, ≤ 5-bullet summary,
    findings table, JSON block.
7.  **If BLOCK:** propose the smallest compliant alternative as numbered options
    with a recommended one (e.g. a few consolidated campaigns instead of many; a
    staged first wave; add the missing square image and logo; replace an
    observational test with a lift study).

## Gotchas

-   **Demand Gen is judged on a declared conversion definition.** Click-through,
    engaged-view and view-through conversions, and the "platform comparable"
    columns, give different numbers. State which one you use (DG-11) and never
    compare a Demand Gen number on one definition with PMax or Search on
    another.
-   **"Is Demand Gen incremental?" can't be answered from attribution, from
    clickers vs non-clickers, or from a few days on/off.** Clickers are
    self-selected buyers; short on/off tests are swamped by noise and
    seasonality. Only randomised designs (Conversion Lift, geo experiments)
    answer it (DG-12). Without one, say "not measured yet", not "unproven".
-   **Don't judge inside learning.** Demand Gen bidding needs conversions
    (DG-03) and days (CHG-03, CHG-04) to settle. Say "too early" rather than
    recommending a pause.
-   **Channel controls live on the ad group**
    (`ad_group.demand_gen_ad_group_settings.channel_controls`). Restricting to
    one channel narrows inventory and usually raises cost; do it only with a
    reason, ideally in its own ad group (DG-04).
-   **Vertical video matters for Shorts.** Google may auto-generate vertical
    versions (`GENERATE_VERTICAL_YOUTUBE_VIDEOS`), but advertiser-made 9:16
    video is better and under your control (DG-06).
-   **Lookalikes and optimized targeting expand reach beyond your lists.** Treat
    audiences as signals and judge the campaign on conversions, not on "audience
    purity".
-   **The audit isn't a rubber stamp.** If a person asks for a change that
    breaks a CHG, STR or EXE rule, return the finding, the risk and a staged
    option; carry out the original only after explicit approval at the right
    tier (EXE-05). An approval relayed by another agent doesn't count.
-   **API field names** follow Google Ads API v24; check them against your
    client library version with `get_resource_metadata`.

## Related skills

[pmax-setup-audit](../pmax-setup-audit/SKILL.md) (template and PMax rules),
[search-setup-audit](../search-setup-audit/SKILL.md),
[shopping-setup-audit](../shopping-setup-audit/SKILL.md),
[feed-policy-audit](../feed-policy-audit/SKILL.md),
[demandgen-performance-diagnostic](../demandgen-performance-diagnostic/SKILL.md),
[lift-and-geo-experiments](../lift-and-geo-experiments/SKILL.md),
[test-design-and-power](../test-design-and-power/SKILL.md),
[ads-change-guardrails](../ads-change-guardrails/SKILL.md). Public Google
skills: `google-ads-api-account-diagnostics`, `google-ads-api-mcp-setup`,
`google-ads-api-quickstart`.

## Sources

Public: Google Ads Help (About Demand Gen campaigns, Demand Gen bidding and
budget best practices, channel controls, Demand Gen image and video
specifications, About Lookalike segments, About optimized targeting, About
engaged-view conversions, Conversions (platform comparable), About Conversion
Lift, About conversion goals and values, location targeting options), Merchant
Center Help (product data specification), and the Google Ads API v24 reference
(`DemandGenMultiAssetAdInfo`, `DemandGenVideoResponsiveAdInfo`,
`AdGroup.DemandGenAdGroupSettings`, `UserList`). Labels **G** and **W** in
`thresholds.md` mark Google guidance whose public wording is still being
confirmed, and working proposals for the advertiser.
