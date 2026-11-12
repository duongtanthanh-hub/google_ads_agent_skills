# 2. Dry run, waves and quota

## Dry run (EXE-01)

Check      | What to verify                                                                                                                              | How to judge
---------- | ------------------------------------------------------------------------------------------------------------------------------------------- | ------------
EXE-01     | Every wave's **exact** payload was sent once with `validate_only = true` through `Mutate` (in chunks ≤ 10,000) and every error was resolved | BLOCK if missing, if run on a sample or an older payload, or if errors remain
Batch jobs | `BatchJobService` has no dry-run flag; validate the same operations with `Mutate` + `validate_only` before uploading them to the job        | BLOCK if the job's operations differ from what was validated
Errors     | Group dry-run errors by error code and field                                                                                                | Fix the generator, not individual rows; rerun the dry run on the fixed payload

A clean dry run proves the API accepts the payload, not that the change is a
good idea (that's [ads-change-guardrails](../../ads-change-guardrails/SKILL.md))
or that policy review will pass (that's the D+1 check).

## Quota forecast (EXE-06)

Count **operations per day** for the job, then add what other jobs use.

```
job_ops/day  = dry-run ops        (plan as counted; G, confirm)
             + write ops
             + retry ops          (≈ expected failure rate × write ops)
             + read requests      (each Search/SearchStream request = 1 op; read-back,
                                   polling and checks)
other_ops/day = scheduled reports + BI pulls + other agents (from logs, peak day)

PASS if job_ops + other_ops ≤ daily limit − EXE-06 headroom
```

Access level (Google Cloud project) | Daily operations             | Planning note
----------------------------------- | ---------------------------- | -------------
Explorer                            | 2,880 on production accounts | Not suitable for bulk jobs
Basic                               | 15,000                       | Small jobs only; spread over days
Standard                            | Unlimited                    | Rate limits still apply; reserve capacity by time window and throttle

**Worked example (Basic access).** Scheduled reports use 3,000 operations a day;
EXE-06 headroom keeps 20% of 15,000 = 3,000 free. Room for the job: 15,000 −
3,000 − 3,000 = 9,000/day. A 40,000-operation listing-group job needs about
80,000 operations (dry run + write), so about 9 days. Suggest consolidating the
change (fewer listing-group operations), or a higher access level, rather than
running it at full speed and starving the reports.

**Standard access.** No daily cap, but rate limits (per account and per project)
return `RESOURCE_TEMPORARILY_EXHAUSTED`. Keep scheduled-report windows clear of
bulk jobs, cap parallel writers, and treat a rising rate of that error as the
signal to slow down. See [ads-api-hygiene](../../ads-api-hygiene/SKILL.md).

BLOCK (EXE-06) if the job is likely to exhaust quota that scheduled jobs need,
or if there is no forecast for a T3 job.

## Wave plan (EXE-04, STR-02)

Check        | What to verify                            | How to judge
------------ | ----------------------------------------- | ------------
EXE-04       | First wave size and spacing               | First wave ≤ 1 account or ≤ 10% of target entities; ≥ 24 h before the next wave; 7 days before any performance-driven decision. BLOCK if everything goes at once across accounts
STR-02       | New campaigns per account per 7 days      | Above max(5, 10% of active) only after a passed pilot wave
Wave content | Which entities go first                   | Pick a representative but low-risk slice (not the top revenue campaigns, not only the easiest)
Labels       | Every wave's entities carry the job label | So a wave can be found, checked and rolled back as a unit (STR-04, OPS-06)

Grow later waves only after a clean gate; roughly doubling or tripling each time
is a common pattern (example, not a threshold).

## Gate between waves

Run after each wave, with the executing agent's results. **All must pass to go
on.**

Check                 | Data                                                                                                                       | Go              | Pause and review           | Halt
--------------------- | -------------------------------------------------------------------------------------------------------------------------- | --------------- | -------------------------- | ----
EXE-03 failure rate   | Failed / submitted operations (count atomic-group failures)                                                                | ≤ 0.5%          | 0.5–2%                     | > 2%
EXE-02 reconciliation | Ledger: succeeded + failed + not executed = submitted; every failure classified (OPS-04)                                   | Complete        | –                          | Any operation unaccounted for
OPS-05 read-back      | Live values vs payload for the read-back sample                                                                            | Match           | –                          | Mismatch not explained
OPS-08 D+1 health     | Disapprovals, serving status, eligibility of wave entities ([ads-post-change-check](../../ads-post-change-check/SKILL.md)) | Clean           | WARN items                 | BLOCK items
EXE-04 spacing        | Time since the wave finished                                                                                               | ≥ 24 h          | –                          | < 24 h
Quota                 | Actual vs forecast operations; quota errors seen                                                                           | Within forecast | Over forecast: re-forecast | Scheduled jobs failed

On **Halt**: stop all remaining waves, list the top error codes with counts, and
propose options (fix and re-validate; roll back the wave from the snapshot; drop
the change). Restarting needs a new approval at T3.
