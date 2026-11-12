---
name: demandgen-performance-diagnostic
description: >-
  Diagnoses why a Google Ads Demand Gen campaign's results changed or look
  weak: falling conversions or value (click, engaged-view and view-through
  conversions reported separately), spend below budget, learning status,
  Maximize clicks vs conversion- or value-based bidding, channel mix (YouTube,
  Shorts, Discover, Gmail, Display, Maps), creative, audiences, and
  product-feed (DEMAND_GEN_PRODUCT_AD) issues.
  Checks measurement first and routes "is Demand Gen incremental?" questions to
  a proper test. Read-only; returns findings in the shared output contract. Use
  for periodic Demand Gen health checks, after a drop, or before a
  pause/scale decision. Don't use for setting up or approving a Demand Gen
  change (use demandgen-setup-audit), for Search, Shopping or PMax (use their
  performance-diagnostic skills), or for running a lift test (use
  lift-and-geo-experiments).
---

# Demand Gen performance diagnostic

Finds the most likely cause of a Demand Gen result, in a fixed order, and says
how sure it is. Builds on the public `google-ads-api-account-diagnostics` skill
(pre-flight, account selection, fallback, period comparison) and adds the Demand
Gen levers that skill doesn't cover.

## When to use

-   "Demand Gen ROAS or conversions are down." "It spends half its budget." "It
    has shown Learning for two weeks." "Video-driven conversions fell."
-   Periodic health check (BI agent at D+7/D+14, or weekly).
-   Before anyone pauses or scales Demand Gen: run this first, then the
    [incrementality routing](references/07_incrementality_routing.md) step.

Not for: approving a planned change (`../demandgen-setup-audit/SKILL.md`,
`../ads-change-guardrails/SKILL.md`), measuring incremental lift
(`../lift-and-geo-experiments/SKILL.md`, `../test-design-and-power/SKILL.md`,
`../causal-impact-analysis/SKILL.md`), or budget planning
(`../budget-seasonality-plan/SKILL.md`).

## Inputs

Input                                     | Required    | Notes
----------------------------------------- | ----------- | -----
Customer ID                               | no          | If missing, select accounts as in [00_preflight.md](references/00_preflight.md)
Question and the date the problem started | yes         | e.g. "value down since Nov 3"
Campaign IDs                              | no          | Default: every Demand Gen campaign with cost in either window, **including paused or removed ones**
Headline KPI and its basis                | yes         | ROAS or CPA, and whether it includes engaged-view conversions (DG-11)
Warehouse / GA4 revenue for the same days | recommended | For the measurement cross-check
`../_shared/client_rules.md`              | yes         | Brand terms, protected products, approvers

## Workflow

1.  **Load the rules.** Read `../_shared/thresholds.md`,
    `../_shared/client_rules.md` and `../_shared/output_contract.md`. Use only
    thresholds from those files (DG- IDs come from `demandgen-setup-audit`,
    MEAS- IDs from `measurement-readiness` and `test-design-and-power`); never
    invent numbers.
2.  **Pre-flight, account selection, fallback.** Follow
    [00_preflight.md](references/00_preflight.md): confirm the MCP tools are
    available, keep enabled non-manager accounts, and validate field names with
    `get_resource_metadata` before querying. If the tools are missing, output
    the plan and GAQL; never probe locally.
3.  **Scope and windows.**
    [01_scope_windows.md](references/01_scope_windows.md): list Demand Gen
    campaigns with cost in either window, whatever their status (Q1); build
    equal, weekday-aligned windows that exclude the conversion lag (DX-01,
    CHG-04); flag sales events and paused days. Apply DX-02 before calling
    anything a change.
4.  **Measurement first** (DX-04 step 1).
    [02_measurement.md](references/02_measurement.md): purchase-type primary
    goal (MEA-01), real values (MEA-02), value-per-conversion anomalies
    (MEAS-01), attribution and lookback windows (MEA-05), offline-upload health,
    and the click / engaged-view / view-through split (DG-11). If measurement is
    broken, say so first: every later finding is low-confidence.
5.  **Delivery, bidding and budget.**
    [03_delivery_bidding.md](references/03_delivery_bidding.md): primary status
    reasons, bid strategy status, Maximize clicks vs conversion vs value bidding
    (BID-01, BID-02), budget headroom (BUD-01, DG-02), conversion volume before
    judging (DG-03, MEA-06).
6.  **Recent changes and learning.**
    [04_changes_learning.md](references/04_changes_learning.md): pull
    `change_event` (≤ 30 days back), line changes up against the drop, and flag
    **edits while learning** as defined there (CHG-01…03, DX-03).
7.  **Channels, creative and audiences.**
    [05_channels_creative_audience.md](references/05_channels_creative_audience.md):
    channel controls vs where spend went, video engagement, ad policy, asset
    performance labels, audience signals and optimized targeting.
8.  **Product feed** (only if the campaign has a Merchant Center link or
    `DEMAND_GEN_PRODUCT_AD` ads).
    [06_product_feed.md](references/06_product_feed.md): FEED-01, FEED-02,
    FEED-04.
9.  **Incrementality routing.** If anyone asks whether Demand Gen "works", is
    "incremental", or should be cut, follow
    [07_incrementality_routing.md](references/07_incrementality_routing.md):
    grade the existing evidence (MEAS-10, DG-12), don't call Demand Gen proven
    or disproven from platform data, and hand over to
    `../lift-and-geo-experiments/SKILL.md`.
10. **Decide and reply** in the output-contract format (see "Output"). Every
    causal statement says measured or inferred, and the window (DX-05).
    Recommendations are suggestions; any write goes to the executing agent with
    an approval tier from EXE-05.

## Output

`../_shared/output_contract.md`, with these diagnostic adaptations:

-   **Verdict:** BLOCK = something must be fixed before any spend decision
    (broken measurement, a MEAS-01 BLOCK, all ads disapproved, no eligible
    products); WARN = issues found; PASS = healthy and no material change
    (DX-02). The approval tier applies to the **recommended actions**.
-   **First summary bullet** names the primary cause per campaign: Tracking,
    Budget-led, Target-led, Learning, Recent change, Channel/inventory,
    Creative, Audience, Feed, Seasonality, or Noise (below DX-02).
-   **Conversion table:** click / engaged-view / view-through in three columns,
    plus the headline basis (DG-11).
-   **JSON:** add `"root_cause"` per campaign, `"conversion_split"`, `"windows"`
    (both date ranges and the excluded lag days) and
    `"evidence": "measured" | "inferred"` per finding.

## Gotchas

-   **Paused campaigns count.** A campaign paused mid-window can explain a drop,
    and a `status = ENABLED` filter hides it. Filter on cost, not status.
-   **Engaged-view conversions are in `metrics.conversions`;** view-through
    conversions aren't (they're in `metrics.view_through_conversions`). Never
    add view-through to the headline unless the client agreed (DG-11).
-   **Attribution-type segments don't mix with cost.** Query the split without
    `metrics.cost_micros`, then compute CPA/ROAS per type in a second step.
-   **Maximize clicks is `TARGET_SPEND` in the API.** Cheap clicks with low
    value per \$1 is what click bidding is built to produce, not proof of a
    creative problem. Check measurement before suggesting value bidding.
-   **Current status isn't the status back then.**
    `bidding_strategy_system_status` shows today's state; whether an edit
    happened "while learning" is inferred from the change timeline (label it
    inferred).
-   **Platform-attributed conversions aren't incrementality.** Neither are
    clicker vs non-clicker comparisons or 4-day on/off tests (MEAS-10).
-   **Don't judge inside the lag or learning window** (CHG-04, DX-03, DG-03).
    Say "too early" instead of recommending a pause.
-   Field names follow Google Ads API v24; confirm with `get_resource_metadata`.

## Sources

Public: Google Ads Help (About Demand Gen campaigns; bid strategies for Demand
Gen; About engaged-view conversions; segmenting by ad event type; conversion
goals, values, attribution and conversion windows; bid strategy statuses;
seasonality adjustments and data exclusions; About Conversion Lift), Merchant
Center Help (product data specification, feed labels), Google Ads API developer
docs (GAQL, `change_event` limits, `shopping_product`,
`ad_group_ad_asset_view`), and the open-source
`google-ads-api-account-diagnostics`, `google-ads-api-mcp-setup` and
`google-ads-api-quickstart` skills (google/skills). Labels **G** and **W** in
`thresholds.md` mark guidance still to confirm and working proposals.
