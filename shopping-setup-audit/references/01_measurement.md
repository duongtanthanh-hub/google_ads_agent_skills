# 1. Measurement

Bidding and reporting only see what is measured. Check this first: if it fails,
every later finding is less reliable.

Check  | What to verify                                                    | Data                                                                                                   | How to judge
------ | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ | ------------
MEA-01 | Biddable goals are purchase-type; micro-conversions are secondary | `campaign_conversion_goal.biddable` by category; `conversion_action.category`, `primary_for_goal` (Q3) | BLOCK if no biddable purchase goal on a value- or conversion-based campaign. WARN if `PAGE_VIEW`, `ADD_TO_CART`, `BEGIN_CHECKOUT`, `ENGAGEMENT` or `DEFAULT` actions are biddable next to purchase. On click-based campaigns the goal still decides what the "Conversions" column reports, so the same rule applies
MEA-02 | Purchase actions send real values                                 | `conversion_action.value_settings.always_use_default_value`, `default_value`                           | WARN if `always_use_default_value = true` on a purchase action; BLOCK if value-based bidding relies on an action without values
MEA-03 | Enhanced conversions for web                                      | Google Ads UI / tag settings                                                                           | WARN if off or unknown; NOT_ASSESSED if it can't be checked
MEA-04 | Consent mode v2 for EEA/UK traffic                                | Target countries (UK, DE, FR, IT, ES, PL in the advertiser's set); client confirmation                 | WARN if EEA/UK is targeted and it's not confirmed
MEA-05 | Data-driven attribution                                           | `conversion_action.attribution_model_settings.attribution_model`                                       | WARN if not data-driven
MEA-06 | Enough conversions to use a target                                | 30-day primary conversions of the campaign, or of the comparable campaigns it replaces                 | PASS ≥ 30; WARN 15–29; below 15, launch Maximize conversion value without a target and say so
SHP-06 | Value per conversion is plausible for a purchase                  | 30-day `conversions_value / conversions` of the campaign (Q3b) vs the AOV input                        | PASS within the SHP-06 band. WARN outside it. BLOCK when a budget, bid or scaling decision relies on a campaign whose value per conversion is below the SHP-06 BLOCK floor

**How to read a low value per conversion (SHP-06)**

1.  Break conversions down by `segments.conversion_action_category` and
    `segments.conversion_action_name` (Q3b; no `metrics.cost_micros` in that
    query). If page views, add-to-carts or a `DEFAULT` action make up the count,
    the campaign is counting non-purchase actions: fix the goal (MEA-01) before
    judging the campaign.
2.  If only purchases are counted, compare with warehouse orders for the same
    days. A gap points at value tagging (currency, single item value instead of
    order value, a default value).
3.  Until fixed, report the campaign's ROAS as "not comparable" and don't use it
    to justify spend.

**Common pattern**

-   One "All products" campaign reported conversions worth less than \$1 each
    while others reported \$15–21. That pattern almost always means non-purchase
    actions are being counted. Treat it as a measurement finding first, not a
    performance win or loss.
-   Record which revenue source is the reference (Google Ads, GA4 or the
    warehouse), so the agent doesn't optimise to one number and report another.
