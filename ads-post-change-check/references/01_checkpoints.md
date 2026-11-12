# 1. Checkpoints

"Changed scope" = the entities in the change record. All windows are in the
account's time zone. Queries are in [queries.md](queries.md).

## D+1: did it apply, and is it serving? (no performance judgement)

Check        | What to verify                        | Data                                                                                                                   | How to judge
------------ | ------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- | ------------
EXE-03       | Write failure rate                    | Per-operation results                                                                                                  | ≤ 0.5% PASS; 0.5–2% WARN; > 2% BLOCK
OPS-05       | Read-back: live values = payload      | Q1 (campaign values), Q2 (asset groups)                                                                                | Any unexplained mismatch = BLOCK; check Q6 for another writer
OPS-08       | Serving status                        | `campaign.primary_status`, `primary_status_reasons`; `asset_group.primary_status`; impressions since go-live (Q3)      | BLOCK if an entity that should serve is `NOT_ELIGIBLE`, `MISCONFIGURED`, `PAUSED` or `REMOVED` unexpectedly, or has 0 impressions 24 h after go-live while it served before
OPS-08       | Policy                                | `ad_group_ad.policy_summary.approval_status`, `asset_group_asset.policy_summary.approval_status`, `review_status` (Q2) | New `DISAPPROVED` = BLOCK; `APPROVED_LIMITED` or still `REVIEW_IN_PROGRESS` = WARN
OPS-08       | Product eligibility (retail)          | `shopping_product.status` for the campaign (see the setup-audit skills)                                                | Use FEED-02 bands on the changed scope
Bid strategy | Status after the change               | `campaign.bidding_strategy_system_status`                                                                              | `LEARNING_*` after a composition or strategy change is expected (note it); `MISCONFIGURED_*` = BLOCK; `LIMITED_BY_*` = WARN with the reason
Spend sanity | Cost on day 1 vs the new daily budget | Q3                                                                                                                     | Up to 2× the daily budget on a day is normal Google pacing; flag only spend far above that or no spend at all

## D+3: is it delivering as expected? (still no ROAS verdict)

Check       | What to verify                                                         | Data                                                                                                                                                              | How to judge
----------- | ---------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Delivery    | Impressions, clicks, cost per day vs the same weekdays in the baseline | Q3, Q4                                                                                                                                                            | Note direction and size; for a budget increase, did spend rise? If not, the budget wasn't the limit (finding, not failure)
Constraints | Budget-limited or bid-limited?                                         | `primary_status_reasons` (`BUDGET_CONSTRAINED`, `BIDDING_STRATEGY_LIMITED`), `bidding_strategy_system_status` (`LIMITED_BY_BUDGET`, `LIMITED_BY_CPC_BID_CEILING`) | Explain; suggest the fix as an option
OPS-01 harm | Tracking broken or wrong traffic                                       | Conversions by conversion date vs clicks (Q5) and conversion action mix                                                                                           | Purchases ≈ 0 while clicks are normal → BLOCK (tracking or landing page); propose fix or rollback
CHG-04      | Performance on immature days                                           | –                                                                                                                                                                 | ROAS/CPA checks = NOT_ASSESSED, "too early (CHG-04)"
OPS-08      | Anything still under review or limited from D+1                        | As D+1                                                                                                                                                            | Still not resolved after D+1 → WARN with owner

## D+7: first performance read

Check   | What to verify                           | Data                                                                             | How to judge
------- | ---------------------------------------- | -------------------------------------------------------------------------------- | ------------
DX-01   | Windows comparable                       | Post days 1–7 (as of D+7) vs pre days −7…−1 (as of D0 snapshot); seasonal events | Equal length, same weekdays, same maturity; events flagged. Without a D0 snapshot, compare only lag-excluded days and say so
DX-02   | Material?                                | Δ cost, conversions, value, ROAS; base-period volume                             | Below the volume floor → "noise / too little data", no rollback on performance
OPS-07  | Rollback trigger                         | ROAS (or CPA) Δ vs baseline, or vs the stop rule in the approval                 | Keep / hold / propose rollback per OPS-07
Control | Same Δ in untouched comparable campaigns | Q4 on control IDs                                                                | Report the change's Δ minus the control's Δ as "inferred" (DX-05)
DX-05   | Measured vs inferred                     | –                                                                                | Every causal statement labelled, with its window

## D+14: decision checkpoint

Check     | What to verify                                                                                                  | Data                                         | How to judge
--------- | --------------------------------------------------------------------------------------------------------------- | -------------------------------------------- | ------------
DX-01     | Post days 1–14 vs pre days −14…−1 at equal maturity; also post days 1–7 now fully mature vs the mature baseline | Q3, Q4, D0 snapshot                          | As D+7
OPS-07    | Rollback trigger on the 14-day comparison                                                                       | As D+7                                       | As D+7; D+14 overrides D+7 when they disagree
DX-03     | Structural follow-ups allowed?                                                                                  | Days since the change                        | ≥ 14 days and outside the lag window → structural recommendations may be made
Ladder    | Next step of a staged plan                                                                                      | Approval's stop rule; all checkpoints so far | See "Staged plans" in 03_rollback_and_stop_rules.md
Close-out | Final report and audit trail                                                                                    | –                                            | Summary of D+1…D+14 with the decision; close the change ID

## Expected direction by change type (sanity check)

Change              | Usually expected                                              | Red flag
------------------- | ------------------------------------------------------------- | --------
Budget up           | Cost up, conversions up, ROAS flat or somewhat down           | Cost flat (budget not binding), or ROAS falling past OPS-07
Budget down         | Cost down, ROAS flat or up                                    | Conversions fall more than cost
tROAS up (stricter) | Cost and volume down, ROAS up                                 | Spend collapses (BID-02 risk)
tROAS down (looser) | Volume and cost up, ROAS down                                 | ROAS falls far below the new target
Strategy switch     | `LEARNING_*` status for up to about 7 days, noisy days        | `MISCONFIGURED_*`, or no conversions with normal clicks
Pause               | Spend stops in scope; other campaigns may pick up some demand | Total account conversions fall more than the paused scope's share
Negatives added     | Fewer irrelevant queries; similar conversions                 | Conversions drop on terms the negatives matched
