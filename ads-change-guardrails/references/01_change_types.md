# 1. Checklists by change type

Run the block for the change type, then the "Every change" block. Use the
current values and change history from [queries.md](queries.md). Percent
change = (new − old) / old.

## Budget (campaign budget amount)

Check        | What to verify                                                        | Data                                                                                                                | How to judge
------------ | --------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- | ------------
CHG-01       | Size of the step and time since the last budget change on that budget | Old vs new `campaign_budget.amount_micros`; last `CAMPAIGN_BUDGET` change in `change_event` (Q2)                    | ≤ 20% and ≥ 3 days = PASS; 20–50% or < 3 days = WARN; > 50% = BLOCK unless an EXE-05 approval names the risk. Applies to increases and cuts
BUD-01       | New budget vs cost per conversion                                     | New budget vs target CPA, or 30-day cost / conversions (Q3)                                                         | WARN if < 3× (common after cuts)
Blast radius | Shared budget                                                         | `campaign_budget.explicitly_shared`, `campaign_budget.reference_count` (Q1)                                         | Count every campaign on the budget toward EXE-05
Context      | Is budget the constraint?                                             | `campaign.primary_status_reasons` contains `BUDGET_CONSTRAINED`; `metrics.search_budget_lost_impression_share` (Q3) | An increase on a campaign that isn't budget-limited usually changes little; say so

**Expected risk of a big increase:** spend rises at once (up to 2× the new daily
budget on a given day), the extra spend usually buys lower-value traffic, so
ROAS often falls before it settles. **Big cut:** lost volume on the best days,
and the strategy may under-deliver for a while.

## Target (tROAS / tCPA, campaign or portfolio)

Check        | What to verify                                  | Data                                                                                                  | How to judge
------------ | ----------------------------------------------- | ----------------------------------------------------------------------------------------------------- | ------------
CHG-02       | Step size and time since the last target change | Old vs new target (Q1, Q4); last change in `change_event` with `changed_fields` containing the target | ≤ 15% and ≥ 7 days = PASS; 15–30% or < 7 days = WARN; > 30% = BLOCK without EXE-05 approval
BID-02       | New target vs 30-day actual                     | New tROAS vs 30-day ROAS of the campaign or comparable non-brand campaigns                            | > 50% stricter = BLOCK (likely to stop serving)
MEA-06       | Enough conversions for a target                 | 30-day conversions (Q3)                                                                               | Below 15: suggest removing the target instead
Blast radius | Portfolio strategy                              | `bidding_strategy.campaign_count` (Q4)                                                                | Count every campaign in the portfolio

## Bid strategy switch

Check  | What to verify                                  | Data                                                               | How to judge
------ | ----------------------------------------------- | ------------------------------------------------------------------ | ------------
CHG-03 | Time since launch or the last structural change | `campaign.start_date_time`; last strategy change in `change_event` | < 7 days BLOCK, 7–13 WARN, ≥ 14 PASS; a strategy switch is structural
BID-01 | Value-based when values are tracked             | New `bidding_strategy_type`                                        | Moving away from `MAXIMIZE_CONVERSION_VALUE` / `TARGET_ROAS` to `TARGET_SPEND` (Maximize clicks) or `MAXIMIZE_CONVERSIONS` on a purchase goal with values = WARN, with the reason asked
MEA-06 | Volume for a target                             | 30-day conversions                                                 | As above
CHG-05 | "Does strategy X work better?"                  | The question behind the switch                                     | Suggest a campaign experiment (≥ 4 weeks) instead of switching and comparing before/after

**The advertiser note:** Maximize clicks with CPC caps of a few cents optimises
for cheap clicks, not sales; in the agent-created campaigns it returned about
half the value per \$1 of the tROAS Shopping and PMax campaigns. Treat it as a
WARN with this evidence, not a ban.

## Status: pause, enable, remove

Check          | What to verify                                          | Data                                                                                          | How to judge
-------------- | ------------------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------
CHG-03         | Days since launch or the last structural change         | `campaign.start_date_time`, `change_event` (status changes), `change_status` for asset groups | < 7 days BLOCK, 7–13 WARN, ≥ 14 PASS, unless OPS-01 applies
CHG-04 / DX-03 | Is the reason a performance judgement on immature data? | Reason given; days of data outside the lag window                                             | "ROAS low after 3 days" = too early; say so
OPS-01         | Emergency exception                                     | Evidence of harm: policy/legal issue, broken tracking, wrong products or URLs serving         | If shown, CHG timing doesn't block; approval still needed
OPS-02         | Irreversible?                                           | Operation is remove (status `REMOVED`)                                                        | Prefer pause; remove needs a snapshot and ≥ T2
Enable         | Re-enabling something paused                            | Why it was paused; is the cause fixed?                                                        | Run the matching setup audit in `live` mode first

## Structure and composition

Adding or removing campaigns, ad groups, asset groups or listing groups; moving
campaigns into or out of a portfolio strategy; splitting or merging.

Check  | What to verify                         | Data                                                                  | How to judge
------ | -------------------------------------- | --------------------------------------------------------------------- | ------------
CHG-03 | Timing                                 | As above                                                              | Composition changes start the Learning status (up to about 7 days)
STR-01 | Projected conversions per new campaign | Historical 30-day conversions for its products and countries          | < 15 BLOCK: suggest an asset group or listing-group split instead
STR-02 | New campaigns per account per 7 days   | New campaigns in the last 7 days (Q6) + this change; active campaigns | Above max(5, 10% of active) without a passed pilot = BLOCK
STR-03 | Product overlap                        | Same products, same country, other campaigns                          | WARN unless documented
STR-04 | Label and naming                       | Planned name and label                                                | WARN if missing. Asset groups can't carry labels in API v24; use the naming convention
Setup  | Full configuration                     | –                                                                     | For new campaigns, also run the setup-audit skill for that campaign type

## Negative keywords and exclusions

Check            | What to verify                            | Data                                                                                                | How to judge
---------------- | ----------------------------------------- | --------------------------------------------------------------------------------------------------- | ------------
STE-04           | No brand term, no protected product term  | Each candidate vs `client_rules.md` (brand list, protected terms), token by token and as substrings | Any hit = BLOCK
Converting terms | Would the negative block terms that sell? | 30-day converting search terms (Q7) matched against the negative with its match type                | WARN, listing the converting terms it would block
Client rule      | Daily human review of candidate negatives | Was today's list reviewed by the named person?                                                      | Not reviewed = not approved yet
Blast radius     | Shared or account-level list              | `shared_set.reference_count` (Q5)                                                                   | Count every campaign the list applies to

## Every change

Check  | What to verify                              | How to judge
------ | ------------------------------------------- | ------------
EXE-01 | `validate_only` result of the exact payload | BLOCK if missing or with unresolved errors. A changed payload needs a new dry run
EXE-05 | Tier for each option                        | See [02_approval_tiers.md](02_approval_tiers.md)
EXE-08 | Post-change checks scheduled                | BLOCK for T2/T3 without a D+1/3/7/14 schedule
OPS-03 | Rollback snapshot                           | BLOCK for T2/T3 without a saved snapshot of every field touched
CHG-04 | Data window behind the request              | Judgement on < 3 days of data = BLOCK as a reason for a performance-driven change
