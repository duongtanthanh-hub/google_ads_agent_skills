# 3. Rollback triggers, stop rules and options

## Rollback triggers

Trigger                        | When it can fire           | Check                                                          | Proposed action (option 1)
------------------------------ | -------------------------- | -------------------------------------------------------------- | --------------------------
Write failures                 | D+1 / after each wave      | EXE-03 > 2%                                                    | Halt further waves; fix and re-run failed operations, or roll back the wave
Wrong state live               | D+1                        | OPS-05 mismatch with no explanation                            | Restore from the snapshot (OPS-03) or re-apply the intended values
Serving stopped or disapproved | Any checkpoint             | OPS-08 BLOCK                                                   | Fix the cause (policy, feed, settings); roll back if the change caused it and the fix isn't quick
Active harm                    | Any checkpoint             | OPS-01 (broken tracking, wrong products or URLs, policy/legal) | Pause or restore at once (still with approval at its tier, asked for urgently in the thread)
Performance                    | D+7 and D+14 only          | OPS-07 BLOCK band on a material Δ (DX-02), at equal maturity   | Restore the previous values from the snapshot
Stop rule in the approval      | As written in the approval | The approver's own rule                                        | As written. A stop rule written in the approval replaces the OPS-07 bands for that change (e.g. a looser ROAS band for a deliberate scale-up); without one, OPS-07 applies

**No performance-based rollback before D+7**, and none on a Δ below the DX-02
floor (CHG-04, DX-03). Earlier rollbacks need a non-performance reason from the
rows above.

## Stop rules (what the agent must not do while a change is being checked)

1.  **No further steps** of a staged plan while any checkpoint has an open WARN
    or BLOCK.
2.  **No escalation on immature data**: no extra budget, looser target or new
    campaigns justified by D+1/D+3 results.
3.  **No overlapping changes** to the same scope during D+0…D+7 unless they fix
    serving or policy (they would confound the read). If one is needed, record
    it as a confounder.
4.  **No silent or automatic rollback.** Propose; execute only after approval.
5.  **No quiet "fix-forward"**: if the change didn't work, say so, then offer
    options; don't stack new changes to hide it.

## Staged plans (ladders and waves)

-   Each step is a new change: guardrail check, approval at its tier, its own
    D+1 check.
-   A next step can be proposed when the CHG spacing for that change type has
    passed (CHG-01, CHG-02) **and** the latest checkpoint has no open WARN or
    BLOCK.
-   Performance checkpoints (D+7, D+14) for a ladder are measured from the first
    step against the pre-ladder baseline. If OPS-07 reaches its WARN band, hold
    the ladder; at its BLOCK band, propose returning to the last good step.
-   Bulk waves follow the gate in
    [ads-bulk-change-protocol](../../ads-bulk-change-protocol/SKILL.md).

## What "rollback" means by change type

Change                                   | Rollback                                                | Notes
---------------------------------------- | ------------------------------------------------------- | -----
Budget                                   | Restore `amount_micros` from the snapshot               | A budget cut back counts as a budget step (CHG-01) but is allowed under OPS-01 when it stops harm
Target                                   | Restore the previous target                             | Same for portfolio strategies (all campaigns in it move)
Bid strategy                             | Switch back to the previous strategy and target         | Structural (CHG-03): the Learning status starts again
Pause                                    | Re-enable                                               | Check nothing else changed while paused
Enable                                   | Pause again                                             | –
Remove                                   | Not possible: recreate from the snapshot                | History stays with the removed entity (OPS-02)
Negatives                                | Remove the added negatives (or the list link)           | Check converting terms recover at D+7
Structure (new campaigns / asset groups) | Pause the new entities; restore old listing-group trees | Don't remove until the decision is final

A rollback is itself a change: run it through
[ads-change-guardrails](../../ads-change-guardrails/SKILL.md) (an exact restore
from the snapshot usually passes as T1/T2) and give it its own D+1 check.

## Report format (same thread as the approval)

1.  Verdict line with the checkpoint, e.g. `VERDICT: BLOCK · scope: change <id>
    · checkpoint: D+1 · approval tier: T1 (for the proposed rollback)`.
2.  ≤ 5 bullets: what happened, the biggest risk, the proposed action.
3.  Findings table (output contract), with NOT_ASSESSED rows for checks that are
    too early.
4.  Numbered options when anything is WARN or BLOCK, e.g.
    1.  (Recommended) Roll back to the snapshot values. Tier T1.
    2.  Keep, fix the cause (e.g. the disapproved assets), recheck at D+3. Tier
        T1.
    3.  Keep as is and continue the plan. Requires accepting the finding at the
        tier in EXE-05.
5.  "I'll execute option N only after explicit approval from <role> in this
    thread." and the date of the next checkpoint.
6.  JSON block, adding `"checkpoint"`, `"change_id"`,
    `"baseline": {"type": "d0_snapshot" | "lag_excluded", "window": "..."}`,
    `"confounders": [...]` and `"options": [...]`.
