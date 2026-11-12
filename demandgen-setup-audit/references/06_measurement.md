# 6. Conversions and measurement

Bidding only optimises toward what is measured, and Demand Gen numbers depend
heavily on which conversions you count. Settle the definition before launch.

Check  | What to verify                                                 | Data                                                                                                                                                                                                                                                                                                                                                                                       | How to judge
------ | -------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------
MEA-01 | Purchase is the biddable goal; micro-conversions are secondary | `campaign_conversion_goal.biddable` (+ `category`, `origin`); `conversion_action.primary_for_goal`, `category`                                                                                                                                                                                                                                                                             | BLOCK if no biddable purchase goal. WARN if `PAGE_VIEW`, `ADD_TO_CART`, `BEGIN_CHECKOUT` or `ENGAGEMENT` is biddable next to purchase
MEA-02 | Purchase sends real values                                     | `conversion_action.value_settings.always_use_default_value`, `default_value`                                                                                                                                                                                                                                                                                                               | WARN if default value always used; BLOCK if value bidding relies on an action with no value
MEA-03 | Enhanced conversions for web                                   | UI / tag settings                                                                                                                                                                                                                                                                                                                                                                          | WARN if off or unknown; NOT_ASSESSED if it can't be checked
MEA-04 | Consent mode v2 when serving EEA/UK                            | Target locations; client confirmation                                                                                                                                                                                                                                                                                                                                                      | WARN if EEA/UK targeted and not confirmed
MEA-05 | Data-driven attribution                                        | `conversion_action.attribution_model_settings.attribution_model`                                                                                                                                                                                                                                                                                                                           | WARN if not data-driven
DG-11  | Declared conversion definition                                 | Plan or reporting spec: actions counted, attribution events (`segments.conversion_attribution_event_type`: `INTERACTION`, `ENGAGED_VIEW`, `IMPRESSION`), lookback windows (`conversion_action.click_through_lookback_window_days`, `view_through_lookback_window_days`; engaged-view window from the UI), column used (`metrics.conversions` vs `metrics.platform_comparable_conversions`) | PASS: written down and the same definition is used for every channel it's compared with. WARN: not declared, or Demand Gen on one definition is compared with PMax/Search on another
DG-12  | Incrementality plan for budget decisions                       | Measurement plan: design, randomisation unit, primary metric, duration, minimum detectable effect, power                                                                                                                                                                                                                                                                                   | PASS: a randomised design (Conversion Lift by users or geography, or a geo experiment) is planned or running before a scale/stop decision. WARN: a budget decision is planned with no such design. BLOCK: a scale or stop decision justified only by an observational comparison
CHG-05 | Experiments run long enough                                    | Planned duration                                                                                                                                                                                                                                                                                                                                                                           | ≥ 4 weeks (aim for 6), first 7 days ignored = PASS; 2–4 weeks WARN; < 2 weeks BLOCK

**Attribution events (for DG-11)**

-   *Click-through* (`INTERACTION`): conversion after a click.
-   *Engaged-view* (`ENGAGED_VIEW`): conversion after someone watched a video ad
    long enough to count as engaged (about 10 seconds, or the whole ad if
    shorter), without clicking. Window is set per account/action in the UI.
-   *View-through* (`IMPRESSION`): conversion after an impression with no click
    or engagement. `campaign.view_through_conversion_optimization_enabled` shows
    whether bidding uses them (only allowed for some Demand Gen campaigns).
-   *Platform comparable* columns (`metrics.platform_comparable_conversions`,
    `_value`) look at Demand Gen on its own and give it fuller credit, closer to
    how social platforms report. Use them to compare with social platforms, not
    with PMax or Search. Treat them as a reporting view: don't assume bidding
    optimises to them (confirm in Google Ads Help, "Conversions (platform
    comparable)", G).

**What doesn't measure incrementality (for DG-12)**

-   Clickers vs non-clickers, or viewers vs non-viewers: self-selected groups;
    people who click were already more likely to buy.
-   Before/after or on/off for a few days: swamped by day-of-week, promotions,
    seasonality and conversion lag (CHG-04, DX-01).
-   Holdouts that aren't randomised (e.g. "markets we didn't pick"), or more
    than one change at once.
-   Attributed conversions or ROAS alone: they show credit, not causation.

These can describe what happened; they can't justify scaling up or turning
Demand Gen off. Design the test with
[test-design-and-power](../../test-design-and-power/SKILL.md) and run it with
[lift-and-geo-experiments](../../lift-and-geo-experiments/SKILL.md).

**Common pattern**

-   The open question is whether Demand Gen adds sales on top of PMax and
    Shopping. Answer it with one well-powered randomised test per main market
    cluster (e.g. US), not with several short tests.
-   Peak season raises baseline demand in both arms; a randomised design still
    works, but say the result applies to peak conditions and avoid launching the
    test in the middle of Black Friday week.
-   Confirm Conversion Lift eligibility and minimum budgets in the Google Ads UI
    or with the Google account team before promising a test.
