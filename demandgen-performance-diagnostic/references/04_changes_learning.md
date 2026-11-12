# 4. Recent changes and learning

Pull every change in the 30 days around the drop (Q8) and line it up with the
daily series (Q1). Most sudden Demand Gen changes follow an edit.

Check    | What to verify                                          | Data                                                                                                                                                   | How to judge
-------- | ------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------
Timeline | Changes on the day the metric moved, or 1–3 days before | Q8: `change_event.change_date_time`, `change_resource_type`, `resource_change_operation`, `changed_fields`, `client_type`, `campaign`, `ad_group`      | List changes by date next to daily cost, conversions and value. Name the change that best lines up and label it **inferred** unless an experiment isolates it
Who      | People vs automation vs agents                          | `change_event.client_type` (`GOOGLE_ADS_WEB_CLIENT`, `GOOGLE_ADS_API`, `GOOGLE_ADS_RECOMMENDATIONS`, `GOOGLE_ADS_AUTOMATED_RULE`, …)                   | Group agent writes (API) separately so the Marketer agent can see its own effect. Don't select `user_email` unless needed
CHG-01   | Budget steps                                            | `CAMPAIGN_BUDGET` updates: old vs new `amount_micros` (from `old_resource` / `new_resource`), spacing                                                  | Flag steps outside the CHG-01 PASS band (size or spacing)
CHG-02   | Target steps                                            | `CAMPAIGN` updates touching `target_cpa_micros` / `target_roas`                                                                                        | Flag steps outside the CHG-02 PASS band (size or spacing)
CHG-03   | Structural changes                                      | Bid-strategy switches, conversion-goal changes, channel-control changes, audiences added or removed, ads created/removed/paused, campaign pause/enable | Flag any inside the CHG-03 window (after launch or the previous structural change)
DX-03    | Enough time since the last structural change to judge   | Date of last structural change                                                                                                                         | In the DX-03 BLOCK band: "too early", no structural recommendation

## Definition: "edit while learning"

An edit counts as **made while learning** when all of these are true:

1.  It's a budget, target, bid-strategy, conversion-goal, channel, audience or
    ad-composition change on a Demand Gen campaign or its ad groups.
2.  It happened (a) inside the CHG-03 window after the campaign's launch or the
    previous structural change, **or** (b) while the campaign was in a learning
    state — `BIDDING_STRATEGY_LEARNING` / `LEARNING_*` now, or inferred from the
    timeline (a structural change inside the CHG-03 BLOCK band, i.e. the
    Learning-status period, before it).
3.  It breaks spacing: a budget change closer to the last one than CHG-01
    allows, a target change closer than CHG-02 allows, or another structural
    change inside the CHG-03 window.

Report each as: date, entity, change, which CHG rule it breaks, and the learning
evidence (`measured` if the status was captured at the time, e.g. in a stored
D+1 check; otherwise `inferred`). Repeated edits while learning are a cause on
their own: the strategy never settles.

## Common patterns to look for

-   Two budget changes closer than CHG-01 spacing on many campaigns at once
    (agent loops). Report the count and suggest a single change per CHG-01 with
    a D+3/D+7 check (EXE-08).
-   Campaigns paused within a week of launch. A pause inside the learning window
    means the campaign was never judged fairly; say so, but don't recommend
    re-enabling without a plan (DG-03 volume, budget DG-02).
-   Many campaigns created in one day (STR-02): learning restarts everywhere at
    once, and daily data becomes noisy.

## Limits

-   `change_event` covers the last 30 days only, needs a date filter and
    `LIMIT ≤ 10000`, and can't select metrics. For older history, use the
    agent's own write log.
-   Conversion-action setting changes aren't always visible in `change_event`;
    if the action mix shifted (step 4) with no event, ask who changed the goal.
