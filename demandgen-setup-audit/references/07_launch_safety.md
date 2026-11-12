# 7. Launch safety and change execution

Applies to every write the audit approves: new campaigns, ad groups, ads,
audiences, budget or target edits.

Check  | What to verify                                                              | How to judge
------ | --------------------------------------------------------------------------- | ------------
STR-01 | Each new campaign can reach enough volume                                   | Projected primary conversions per 30 days for its products and markets: PASS ≥ 30; WARN 15–29; BLOCK < 15 (consolidate into an existing campaign or ad group)
STR-02 | Launch batch size                                                           | New campaigns in this plan plus the last 7 days vs active campaigns: PASS ≤ max(5, 10% of active); WARN above that after a passed pilot (EXE-04); BLOCK otherwise
STR-03 | Overlap                                                                     | WARN if several Demand Gen campaigns target the same audience and market (they compete for the same users); write down whether the overlap is intended
STR-04 | Labels and naming                                                           | WARN if the agreed label or naming for agent-created entities is missing
EXE-01 | `validate_only` dry run of the exact payload, result attached               | BLOCK if missing or errors unresolved
EXE-02 | Payload size and method                                                     | BLOCK if a synchronous mutate request has > 10,000 operations; larger jobs use BatchJobService or chunking with reconciliation
EXE-03 | Failure rate (post_launch, or after each wave)                              | ≤ 0.5% PASS; 0.5–2% WARN, pause before the next wave; > 2% BLOCK
EXE-04 | Staged rollout                                                              | First wave ≤ 1 account or ≤ 10% of entities; ≥ 24 h before the next; ≥ 7 days before performance decisions. BLOCK if everything goes out at once across accounts
EXE-05 | Approval tier                                                               | Compute T1/T2/T3 from blast radius. BLOCK if the approver is wrong or missing, or the approval was relayed by another agent
EXE-06 | Quota forecast                                                              | BLOCK if the job is likely to starve scheduled reports (keep ≥ 20% headroom)
EXE-07 | Read efficiency                                                             | WARN for repeated full pulls where `change_status` / `change_event` would do
EXE-08 | Post-change checks                                                          | D+1, D+3, D+7, D+14 scheduled; BLOCK for T2/T3 changes without a schedule
CHG-03 | Structural change timing (pause, removal, restructure, bid-strategy switch) | Days since launch or the last structural change: PASS ≥ 14; WARN 7–13; BLOCK < 7. Fixing a serving or policy problem is the exception
CHG-04 | Evaluation window                                                           | Use data older than the conversion lag (default: exclude the last 7 days). WARN if the judgement uses the last 3–7 days; BLOCK if it rests on < 3 days of data. For Demand Gen, engaged-view and view-through conversions arrive later than clicks

**Create paused, enable after review**

1.  Create the campaign, ad groups and ads **paused** (dry run first).
2.  Run this audit in `live` mode on the paused entities: ads approved,
    audiences attached, channels and targeting as planned.
3.  Enable after approval at the right tier.
4.  D+1: `post_launch` audit (serving, policy, spend pacing). Continue with the
    next wave only if EXE-03 passes and nothing is BLOCK.

**Rollback.** Before any T2/T3 write, save the current state of everything it
touches (budgets, targets, statuses, channel controls, audiences) so it can be
restored in one step.

**Common pattern**

-   One Demand Gen campaign per designer, seller or product will fail STR-01 and
    DG-02. Group by category or season, with ad groups per theme.
-   Don't pause a Demand Gen campaign in its first week because attributed ROAS
    looks low; check serving and policy, then wait (CHG-03, DG-03).
