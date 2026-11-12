# 6. Launch safety and change execution

Applies to every write the audit approves: new campaigns, ad groups, product
groups, inventory filters, negatives, bid, budget or status changes.

Check  | What to verify                                                | How to judge
------ | ------------------------------------------------------------- | ------------
EXE-01 | `validate_only` dry run of the exact payload, result attached | BLOCK if missing or errors unresolved
EXE-02 | Payload size and method                                       | BLOCK if a synchronous mutate request has > 10,000 operations. Larger jobs use BatchJobService (always partial-failure) or chunking, with a plan to reconcile failed operations. A product-group tree must be sent whole in one request, so count tree operations per ad group
EXE-03 | Failure rate (`post_launch`, or after each wave)              | ≤ 0.5% PASS; 0.5–2% WARN, pause before the next wave; > 2% BLOCK, halt and investigate the top error codes. For product-group trees, look for the one root error behind many cascading ones
EXE-04 | Staged rollout                                                | First wave ≤ 1 account or ≤ 10% of target entities; ≥ 24 h before the next wave; ≥ 7 days before decisions based on performance. BLOCK if everything goes out at once across accounts
EXE-05 | Approval tier                                                 | Compute T1/T2/T3 from the blast radius. BLOCK if the approver is wrong or missing, or the approval was relayed by another agent
EXE-06 | Quota forecast                                                | Estimated operations vs the daily quota, minus ≥ 20% headroom for scheduled reports. BLOCK if the job is likely to starve scheduled jobs
EXE-07 | Read efficiency of monitoring                                 | WARN for repeated full or hourly pulls where `change_status` / `change_event` incremental sync would do
EXE-08 | Post-change checks                                            | D+1 (errors, policy, serving, products eligible), D+3, D+7, D+14 (vs baseline, outside the conversion lag). BLOCK for T2/T3 changes without a schedule
CHG-03 | Structural change timing                                      | No pausing, removing, restructuring or bid-strategy switch within 14 days of launch or the last structural change, except to fix serving or policy problems. < 7 days = BLOCK
CHG-04 | Evaluation window                                             | Use data older than the conversion lag; say "too early" rather than act on 1–3 days of data
CHG-05 | Testing                                                       | For "does X work?" (e.g. Shopping vs PMax for a category), set up an experiment of ≥ 4 weeks (ignore the first 7 days) instead of a before/after comparison

**Create paused, enable after review**

1.  Create campaigns **paused** (dry run first).
2.  Run this audit in `live` mode on the paused campaigns: products eligible at
    campaign scope, tree valid, negatives compliant.
3.  Enable the first wave only after approval at the right tier.
4.  D+1: `post_launch` audit. Continue only if EXE-03 passes and nothing is
    BLOCK.

**Pausing for performance**

-   A campaign paused inside the CHG-03 window never got a fair read. If many
    new campaigns are paused within days, the problem is the launch design
    (STR-01, SHP-01, SHP-02), not each campaign. Say so instead of approving
    more launch-and-pause cycles.
-   Pausing to fix serving or policy problems (disapprovals, wrong feed label,
    runaway spend against the plan) is always allowed; state the reason.

**Rollback** Before any T2/T3 write, save the current state of everything it
touches (budgets, bid strategies and caps, statuses, product-group trees,
negative lists) so it can be restored in one step.
