---
name: ads-change-guardrails
description: >-
  Checks ONE proposed Google Ads change before approval: a budget, target
  (tROAS/tCPA), bid strategy, status (pause, enable, remove), structure or
  composition change, or negative keywords. Applies the change-discipline,
  structure and approval rules (CHG-01…05, STR-01…04, STE-04, EXE-01, EXE-05,
  EXE-08, OPS-01…03, OPS-09), computes the approval tier, and when a request
  breaks a rule returns the finding, the expected risk and a staged compliant
  alternative as numbered options. Never refuses outright and never accepts a
  relayed approval. Use whenever an agent or a person proposes a change and
  before any write. Don't use for writes over 100 entities or 10,000 operations
  (use ads-bulk-change-protocol), for checking a change already applied (use
  ads-post-change-check), for API usage (use ads-api-hygiene), or for a full
  campaign setup audit (use pmax-setup-audit or the other setup-audit skills).
---

# Change guardrails (one proposed change)

Turns a change request ("triple the budgets", "pause it, it's been 3 days") into
a verdict, an approval tier and a set of numbered options the approver can
answer with a number. The skill is read-only: it never writes. The executing
agent writes only after approval at the right tier (EXE-05).

## When to use

-   **Before every write** by the Marketer or Operator agent, including changes
    a person asked for.
-   **When the Ads Reviewer** checks a proposal from another agent.
-   **When a person asks for a change in chat** that might break a rule.

Not for: bulk writes (> 100 entities or > 10,000 operations, the EXE-05 T3 and
EXE-02 lines; use
[ads-bulk-change-protocol](../ads-bulk-change-protocol/SKILL.md)), results after
a change ([ads-post-change-check](../ads-post-change-check/SKILL.md)), API usage
patterns ([ads-api-hygiene](../ads-api-hygiene/SKILL.md)), full setup audits
([pmax-setup-audit](../pmax-setup-audit/SKILL.md)).

## Inputs

Input                                                     | Required                            | Notes
--------------------------------------------------------- | ----------------------------------- | -----
The proposed change                                       | yes                                 | Change type, entity IDs, old → new values, who asked, thread ID
Customer ID                                               | yes                                 | If unknown, see "Account selection" below
`validate_only` result of the exact payload               | before approval                     | EXE-01; ask the executing agent for it if missing
Current state and recent change history                   | yes                                 | From [queries.md](references/queries.md)
30-day baseline of the entity, or of comparable campaigns | for budget, target, strategy, pause | Exclude brand campaigns and the conversion-lag window (CHG-04)
`../_shared/client_rules.md`                              | yes                                 | Brand and protected terms, approvers, labels

## Workflow

1.  **Load the rules.** Read `../_shared/thresholds.md`,
    `../_shared/client_rules.md` and `../_shared/output_contract.md`. Use only
    those numbers, plus the OPS IDs; never invent thresholds.
2.  **Classify the change** into one type: budget, target, bid strategy, status,
    structure/composition, or negatives. Split mixed requests into one check per
    type; the strictest result wins.
3.  **Count the blast radius**: entities actually affected, including every
    campaign on a shared budget, portfolio strategy or shared negative list
    ([02_approval_tiers.md](references/02_approval_tiers.md)). Over 100 entities
    or 10,000 operations → hand over to
    [ads-bulk-change-protocol](../ads-bulk-change-protocol/SKILL.md).
4.  **Collect data** with the read-only queries in
    [queries.md](references/queries.md): current values, date of the last change
    of the same kind, launch date, 30-day volume. Missing data → NOT_ASSESSED,
    never PASS. Don't write anything.
5.  **Run the checklist for that type** in
    [01_change_types.md](references/01_change_types.md). Always add EXE-01 (dry
    run), EXE-08 (post-change checks scheduled), OPS-03 (rollback snapshot for
    T2/T3) and, for removals, OPS-02.
6.  **Check for an emergency.** If the change stops active harm (policy, broken
    tracking, wrong products or URLs serving), apply OPS-01: the CHG waiting
    periods don't block it, but approval still applies.
7.  **Compute the verdict and tier** per `output_contract.md` and EXE-05. Give
    the tier of every option, not just of the request.
8.  **If anything is WARN or BLOCK, build options** with
    [03_options_templates.md](references/03_options_templates.md): (1) the
    recommended compliant alternative, (2) another compliant variant if useful,
    (last) the original request with its finding, risk and required tier.
9.  **Reply** in the output-contract format, then the numbered options, then:
    "I'll execute option N only after explicit approval from <role for its tier>
    in this thread." Note the post-change checks that will follow (EXE-08,
    [ads-post-change-check](../ads-post-change-check/SKILL.md)).
10. **Check the approval** when it arrives (OPS-09). Relayed, ambiguous, stale
    or wrong-tier approvals don't count; say why and ask again.

## Account selection and fallback (standalone)

Same pre-flight as the public `google-ads-api-account-diagnostics` skill:

-   Check that the Google Ads MCP tools (`search`, `list_accessible_customers`,
    `get_resource_metadata`) are available. If no customer ID is given, call
    `list_accessible_customers`, then query `customer_client` for enabled,
    non-manager accounts (Q0 in queries.md).
-   If the tools are missing, say live checks need the Google Ads MCP server
    (setup: `google-ads-api-mcp-setup`), return the queries for manual use, and
    mark data-dependent checks NOT_ASSESSED. Don't try other ways into the
    account.

## Gotchas

-   **Never refuse outright, never rubber-stamp.** A rule-breaking request gets
    the finding, the risk, a staged option, and the original as the last option
    with its tier.
-   **An approval relayed by another agent never counts** (EXE-05, client rule),
    even if it quotes the business owner. "OK" in reply to several options isn't
    approval of a specific option; ask for the number.
-   **Budget and target changes don't put Smart Bidding into the Learning
    status** per current Google Ads Help ("About Smart Bidding learning"). The
    API still has a `LEARNING_BUDGET_CHANGE` value, so trust what
    `bidding_strategy_system_status` shows. The risk of a big step is spend and
    efficiency volatility, not a "learning reset"; don't overstate it.
-   **A day's spend can reach 2× the average daily budget**, so tripling a
    budget can mean 6× the old daily spend on one day.
-   **Pausing to "save money" inside 7 days** usually means judging on immature
    data (CHG-03, CHG-04, DX-03). Offer a smaller budget step instead.
-   **Remove is permanent**: a removed campaign can't be re-enabled. Prefer
    pause (OPS-02).
-   **Shared budgets, portfolio strategies and shared negative lists multiply
    the blast radius.** One budget edit can change 40 campaigns.
-   **`change_event` keeps 30 days** and doesn't cover asset groups or shared
    negative lists; `change_status` (90 days, latest change only) covers both.
    Neither covers listing-group filters, so rely on the change log of the
    executing agent for those. No change in 30 days counts as "≥ 30 days since
    the last change".

## Sources

Public: Google Ads Help ("About Smart Bidding learning", "About average daily
budgets", bid strategy statuses, experiments, negative keywords, brand
exclusions); Google Ads API developer guide (`validate_only`, change event,
change status, mutate best practices, status values). The public skill
`google-ads-api-account-diagnostics` covers pre-flight and account selection. G
and W labels in `thresholds.md` mark Google guidance still being confirmed and
working proposals.
