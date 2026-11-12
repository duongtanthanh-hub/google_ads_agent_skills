---
name: ads-post-change-check
description: >-
  Checks a Google Ads change after it has been applied, at D+1 (write errors,
  read-back, policy, serving, eligibility), D+3 (delivery and pacing), D+7 and
  D+14 (performance against a pre-change baseline at equal conversion-lag
  maturity), using CHG-04, DX-01…05, EXE-03, EXE-08 and OPS-05, OPS-07, OPS-08,
  OPS-10.
  Applies rollback triggers and stop rules, proposes rollback or the next step
  as numbered options, and reports in the output contract in the same thread
  as the approval. Read-only. Use after every applied T2/T3 change (and T1
  changes when asked), and between waves of a bulk job. Don't use before a
  change (use ads-change-guardrails or ads-bulk-change-protocol), for API usage
  issues (use ads-api-hygiene), or for open-ended performance diagnosis not tied
  to a change (use the performance-diagnostic skills).
---

# Post-change check (D+1, D+3, D+7, D+14)

Answers "did the change do what we expected, and is it safe to continue?" for
one applied change. Early checkpoints look for breakage; later ones look at
performance, but only on data mature enough to judge. The skill never writes:
rollback or the next step is proposed as options and executed by the agent only
after approval (EXE-05).

## When to use

-   **Scheduled** (EXE-08) after each applied change: D+1, D+3, D+7, D+14,
    counted from the time the change went live, in the account's time zone.
-   **Between waves** of a bulk job (gate in
    [ads-bulk-change-protocol](../ads-bulk-change-protocol/SKILL.md)).
-   **On demand** when someone asks "how is the change doing?"

Not for: approving a change
([ads-change-guardrails](../ads-change-guardrails/SKILL.md)), API errors and
quota ([ads-api-hygiene](../ads-api-hygiene/SKILL.md)), broad "why did
performance drop?" questions (performance-diagnostic skills), or measuring
incrementality (media-effectiveness skills).

## Inputs

Input                         | Required           | Notes
----------------------------- | ------------------ | -----
Change record                 | yes                | Change ID, entities, old → new values, time applied, approval tier, thread, stop rule from the approval
Write results                 | D+1                | Per-operation results; failure count
Baseline snapshot taken at D0 | strongly preferred | See [02_baseline_and_lag.md](references/02_baseline_and_lag.md); if missing, the skill builds a lag-excluded baseline and says so
Rollback snapshot             | yes for T2/T3      | OPS-03; needed to propose a one-step rollback
Control campaigns             | recommended        | Comparable campaigns not changed in the window
`../_shared/client_rules.md`  | yes                | Approvers, protected products, intentional exclusions

## Workflow

1.  **Load the rules.** Read `../_shared/thresholds.md`,
    `../_shared/client_rules.md` and `../_shared/output_contract.md`. Use only
    those numbers and the OPS IDs.
2.  **Pre-flight.** As in the public `google-ads-api-account-diagnostics` skill:
    check the MCP tools (`search`, `list_accessible_customers`,
    `get_resource_metadata`), pick the enabled client account. Without tools,
    return the queries and mark checks NOT_ASSESSED.
3.  **Pin the timeline.** Read `customer.time_zone`; take the go-live time from
    the change record or `change_event`; compute which checkpoint is due.
4.  **List confounders.** Other changes to the same scope since D0 (Q6),
    seasonal events in either window (DX-01), tracking changes. Note them; they
    limit what can be concluded (DX-05).
5.  **Run the checkpoint** in [01_checkpoints.md](references/01_checkpoints.md):
    -   **D+1:** read-back (OPS-05), write errors (EXE-03), policy, serving,
        eligibility (OPS-08). No performance judgement.
    -   **D+3:** delivery and pacing vs expectation; bid strategy status; still
        no ROAS verdict (CHG-04), except harm (OPS-01).
    -   **D+7 / D+14:** performance vs baseline at equal maturity (DX-01; lag
        per OPS-10), with the materiality floor (DX-02) and the rollback trigger
        (OPS-07, or the stop rule written in the approval).
6.  **Apply the rules** in
    [03_rollback_and_stop_rules.md](references/03_rollback_and_stop_rules.md):
    keep, hold (stop further steps), or propose rollback.
7.  **Reply in the same thread as the approval**, in the output-contract format
    with the checkpoint in the verdict line (below), then numbered options when
    anything is WARN or BLOCK, then the next checkpoint date.

```
VERDICT: WARN · scope: change <id> (12 campaigns) · checkpoint: D+7 · approval tier: T1 (for the proposed next step)
```

Performance checks that are too early are NOT_ASSESSED with the reason "too
early (CHG-04)", never PASS.

## Gotchas

-   **D+1 and D+3 ROAS is incomplete.** Recent days lack late conversions.
    Comparing them with a mature baseline always shows a "drop". Compare at
    equal maturity or wait.
-   **Black Friday, Cyber Monday and Christmas** sit inside late-November and
    December windows. Flag them (DX-01) and use control campaigns; otherwise the
    season gets credited to the change.
-   **A budget increase that isn't spent** means the budget wasn't the limit;
    that's a finding, not a failure.
-   **Check what is being counted.** If conversion value per conversion falls
    sharply (e.g. under \$1 while others are \$15–21), check the conversion
    actions included before judging the change (DX-04 step 1).
-   **A rollback is a change too**: it gets its own guardrail check, tier and
    post-change checks. Restoring the snapshot exactly is usually T1/T2.
-   **Never roll back silently or automatically.** Propose it; execute only
    after approval in the thread.
-   **Removed entities can't be restored**; "rollback" means recreating them
    from the snapshot.

## Sources

Public: Google Ads Help (bid strategy statuses, "About Smart Bidding learning",
"About average daily budgets", conversion delay / days to conversion, policy and
approval statuses, product status in Merchant Center), Google Ads API developer
guide (primary status, policy summary, change event,
`conversions_by_conversion_date` metrics, conversion lag segment). Labels G and
W in `thresholds.md` mark guidance being confirmed and working proposals.
