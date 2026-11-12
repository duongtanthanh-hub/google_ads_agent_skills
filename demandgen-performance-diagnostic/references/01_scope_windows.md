# 1. Scope and comparison windows

Check        | What to verify                                                                                        | Data                                                                                          | How to judge
------------ | ----------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------
Scope        | Every Demand Gen campaign that spent in **either** window, including ones paused or removed during it | Q1 (no status filter; `metrics.cost_micros > 0`), `campaign.status`, `campaign.end_date_time` | List paused/removed/ended campaigns separately with their last day of spend. A campaign that stopped mid-window is a candidate cause on its own
DX-01        | Windows of equal length, aligned by weekday, same lag maturity                                        | The dates you chose                                                                           | PASS if both windows have the same number of each weekday (use 7, 14 or 28 days, or year-over-year shifted by 364 days) and both exclude the lag. WARN otherwise
CHG-04       | The most recent days are left out                                                                     | Lag from Q5 (`segments.conversion_lag_bucket`)                                                | Leave out the days in which most of the campaign's conversions usually still arrive; if the lag is unknown, use the CHG-04 default. Engaged-view conversions often arrive later than click conversions
DX-02        | The change is material                                                                                | Base-period conversions or cost                                                               | Below the volume floor: report "noise / too little data" and stop the root-cause search for that campaign
Events       | Sales events or peaks inside either window                                                            | Calendar; Q12 (seasonality adjustments, data exclusions)                                      | Flag Black Friday (27 Nov 2026), Cyber Monday (30 Nov 2026), Christmas shipping cut-offs, local holidays per market. Don't compare a peak week with a normal week without saying so
Partial days | Campaign start, pause or end inside a window                                                          | Q1 daily rows                                                                                 | Compare like with like: drop the days a campaign didn't run from both windows, or compare daily averages over days served

## Choosing windows (worked example)

Question asked on Wed 11 Nov 2026: "Demand Gen value is down this week."

-   Lag: 80% of conversions arrive within 5 days → leave out 6–11 Nov.
-   Current window: the latest full weeks before the lag cut-off, Mon 19 Oct –
    Sun 1 Nov (14 days, 2 of each weekday). If the question is about this week,
    report 2–5 Nov separately as a partial week and 6–11 Nov as "immature"
    (inside the lag).
-   Prior window: Mon 5 Oct – Sun 18 Oct (same weekdays, same lag maturity).
-   Year-over-year check (optional): 364 days earlier keeps weekdays aligned.
    Late-November weeks aren't comparable with October weeks; compare them with
    last year's Black Friday weeks instead.

## Seasonality notes

-   **Seasonality adjustments** tell Smart Bidding to expect a short conversion
    rate change (Google recommends them for events of about 1–7 days). If one
    ran inside a window, results in that window aren't comparable. **Data
    exclusions** mark days with broken conversion tracking; if one exists,
    exclude those days from the comparison too.
-   Print-on-demand demand peaks in Q4 (gifts, Christmas, personalised items). A
    November rise or a January fall is expected; say "seasonal (inferred)" only
    with year-over-year or market evidence.
-   In multi-market accounts, check whether one market drives the change by
    comparing campaigns per country (the advertiser splits campaigns by market).
