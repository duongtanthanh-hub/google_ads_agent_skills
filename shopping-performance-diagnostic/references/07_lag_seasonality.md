# 7. Conversion lag and seasonality (window checks; demand part of DX-04)

Check                    | What to verify                              | Data                                                                          | How to judge
------------------------ | ------------------------------------------- | ----------------------------------------------------------------------------- | ------------
DXP-05                   | Lag window                                  | Q11 `segments.conversion_lag_bucket` with `metrics.conversions`, last 90 days | Lag days per DXP-05; exclude them from both windows. Too few conversions (typical for small Shopping campaigns): use the CHG-04 default, or compute lag across all Shopping campaigns in the country
CHG-04                   | No judgement on immature days               | Windows from Workflow step 2                                                  | WARN if the current window includes lag days; "too early" in the CHG-04 BLOCK band
Click vs conversion date | Reporting-date effect                       | Q11b `conversions_value` vs `conversions_value_by_conversion_date` by date    | Value by conversion date flat → the "drop" is lag
DX-01                    | Events in either window                     | Country calendars; promotions                                                 | Flag Black Friday / Cyber Monday, Christmas shipping cut-offs, Halloween, Mother's/Father's Day by country. Unflagged = WARN
Year-ago                 | Same weeks last year                        | Q2 for the aligned weeks last year, if the campaign or a predecessor existed  | Similar move last year → inferred seasonality
DXP-08                   | Seasonality adjustments and data exclusions | Q12                                                                           | Bands per DXP-08. They only affect Smart Bidding (e.g. target ROAS, Maximize conversion value); they don't change Manual CPC or Maximize clicks bids

**Notes**

-   Click-based Shopping campaigns don't anticipate seasonal conversion-rate
    changes; their value per \$1 swings with the season while the clicks they
    buy stay similar.
-   After a country's Christmas shipping cut-off, conversion rate drops; that's
    expected, not a fault.

**Gotchas**

-   `segments.conversion_lag_bucket` can't be selected with
    `metrics.cost_micros`.
-   Lag varies by country and product; personalised items can take longer.
