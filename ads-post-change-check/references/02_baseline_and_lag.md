# 2. Baseline, conversion lag and fair comparisons

## Why equal maturity matters

Conversions are credited to the day of the click (`metrics.conversions`) and
keep arriving for days after it. On D+3, the post-change days are missing many
of their conversions; a fully matured baseline isn't. Comparing them always
shows a fake drop (CHG-04).

## Preferred design: snapshot at D0, compare at equal age

1.  **At go-live (D0)**, save per entity and day: cost, clicks, impressions,
    conversions, conversion value for days −14…−1, **as reported at D0**. The
    newest days are immature, exactly as the post-change days will be.
2.  **At D+7**, compare post days 1–7 (as reported at D+7) with pre days −7…−1
    (as reported at D0). Each post day has the same age as its matching pre day,
    and the weekdays line up.
3.  **At D+14**, compare post days 1–14 (as of D+14) with pre days −14…−1 (as of
    D0). Also compare post days 1–7, now mature, with a mature baseline (days
    −21…−8, read at D+14).
4.  Save the comparison inputs in the change record so the check can be
    repeated.

## Fallback when there is no D0 snapshot

-   Exclude the conversion-lag window from both sides (CHG-04; default the last
    7 days if the lag is unknown). At D+7 that may leave no usable post days:
    then performance is NOT_ASSESSED, "too early".
-   Say in the summary that the baseline was rebuilt and is less reliable.

## Estimating the conversion lag

-   `segments.conversion_lag_bucket` with `metrics.conversions` over a mature
    90-day window (Q5b) shows how many days after the click conversions arrive.
    Set the lag per OPS-10 (the bucket by which the OPS-10 share of conversions
    has arrived; OPS-10 default if unknown). Conversion segments may not combine
    with cost metrics; query them separately.
-   `metrics.conversions_by_conversion_date` (and
    `..._value_by_conversion_date`) credit conversions to the day they happened,
    not the click day. Useful for the D+3 tracking check: if conversions by
    conversion date fall to about zero while clicks are normal, tracking or the
    site is broken, whatever the lag.

## Windows and materiality

-   **DX-01:** equal length, aligned weekdays, same maturity; flag events such
    as Black Friday / Cyber Monday (late November), Christmas and local holidays
    in each market inside either window.
-   **DX-02:** a Δ counts only if it's large **and** the base period has enough
    volume. Below the floor, report "noise / too little data" and don't trigger
    OPS-07 on it. Aggregate small entities (e.g. all campaigns in the change)
    before testing.
-   **DX-05:** label each statement measured or inferred, with the window.

## Control comparison (recommended)

Pick comparable campaigns that the change didn't touch (same country and
campaign type, no changes in the window per `change_event`). Compute the same Δ
for them. Report:

```
change effect (inferred) ≈ Δ changed scope − Δ control
```

This removes much of the seasonality (e.g. a Black Friday lift that hits
everything). It is still an inference (DX-05), not a measured lift; for "does X
work?" questions, use an experiment (CHG-05).

## Check what is being counted (DX-04 step 1)

Before blaming or crediting the change:

-   Break conversions down by `segments.conversion_action` (Q5a, without cost).
    A shift toward cheap non-purchase actions (e.g. value per conversion falling
    from \$15–21 to under \$1) is a measurement problem, not a performance
    result.
-   Confirm the value source used for decisions (Google Ads vs warehouse/GA4) is
    the same as in the baseline.

## Worked example: budget +20% on 12 PMax campaigns, applied 3 Nov

-   D0 snapshot saved for 20 Oct–2 Nov.
-   D+7 (10 Nov): post 3–9 Nov vs pre 27 Oct–2 Nov, both read at the same age.
    Cost +18%, conversion value +9%, ROAS −8% (above the DX-02 floor). Control
    PMax campaigns: ROAS −2%. Inferred effect on ROAS ≈ −6%: within the OPS-07
    "keep" band, so the next +20% step can be proposed (CHG-01 spacing has
    passed; it is a new change with its own guardrail check and D+1 check).
-   D+14 (17 Nov): repeat on 14 days against the same pre-ladder baseline; no
    events in either window yet. Black Friday (27 Nov) will fall inside later
    windows: flag it and lean on the control comparison.
