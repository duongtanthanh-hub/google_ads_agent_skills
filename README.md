# Google Ads agent skills (experimental)

A library of 21 Agent Skills (`SKILL.md` format) for AI agents that **plan,
audit, change and measure Google Ads accounts through the Google Ads API**. They
were written for multi-market ecommerce advertisers whose agents read through
the open-source Google Ads MCP server and write through their own API tooling,
with human approval.

> [!IMPORTANT] **Experimental, unofficial, draft (v0.2).** These skills aren't
> Google policy. Rules are labelled by source: **P** public Google
> documentation, **G** Google guidance whose public wording still has to be
> confirmed, **W** working proposals, **C** client-owned rules. Confirm G rules
> and agree W rules with the advertiser before relying on them. Owner:
> duongtanthanh@.

## Conventions

-   **Shared rules:** every number lives in
    [_shared/thresholds.md](_shared/thresholds.md) as a rule ID (e.g. `CHG-01`,
    `BUD-01`). Skills reference IDs and never hard-code values.
-   **One answer format** for every skill:
    [_shared/output_contract.md](_shared/output_contract.md). It gives a PASS /
    WARN / BLOCK verdict, an approval tier (T1–T3), a findings table with
    evidence, and a JSON block.
-   **Advertiser-owned rules** (brand terms, protected products, intentional
    exclusions, approvers) go in
    [_shared/client_rules.md](_shared/client_rules.md). It's a template; each
    advertiser fills in their own.
-   **Read-only:** skills never write to Google Ads. Writes stay with the
    executing agent after approval.
-   **Field names and tests:** GAQL uses Google Ads API v24 field names. Each
    skill ships `evals/cases.md`, including safety cases and a clean case.
-   **Install the whole folder.** Skills link to `../_shared/`, so installing a
    single skill folder on its own breaks those links.

## Skills

Category            | Skill                                                                         | Purpose
------------------- | ----------------------------------------------------------------------------- | -------
Change safety       | [ads-change-guardrails](ads-change-guardrails/SKILL.md)                       | Check one proposed change before approval; approval tier; staged alternatives
                    | [ads-bulk-change-protocol](ads-bulk-change-protocol/SKILL.md)                 | Bulk writes: dry run, batch jobs, partial failures, waves, failure caps, rollback
                    | [ads-api-hygiene](ads-api-hygiene/SKILL.md)                                   | Efficient, traceable API use: SearchStream, incremental sync, quota, errors
                    | [ads-post-change-check](ads-post-change-check/SKILL.md)                       | D+1/3/7/14 checks after a change; rollback triggers
Setup audit         | [pmax-setup-audit](pmax-setup-audit/SKILL.md)                                 | Performance Max setup
                    | [shopping-setup-audit](shopping-setup-audit/SKILL.md)                         | Standard Shopping setup
                    | [feed-policy-audit](feed-policy-audit/SKILL.md)                               | Merchant Center feed and policy health
                    | [demandgen-setup-audit](demandgen-setup-audit/SKILL.md)                       | Demand Gen setup
                    | [search-setup-audit](search-setup-audit/SKILL.md)                             | Search setup, incl. AI Max
Diagnostics         | [pmax-performance-diagnostic](pmax-performance-diagnostic/SKILL.md)           | Why PMax results changed
                    | [shopping-performance-diagnostic](shopping-performance-diagnostic/SKILL.md)   | Why Shopping results changed
                    | [demandgen-performance-diagnostic](demandgen-performance-diagnostic/SKILL.md) | Why Demand Gen results changed; routing to lift tests
                    | [search-performance-diagnostic](search-performance-diagnostic/SKILL.md)       | Why Search results changed
Media effectiveness | [measurement-readiness](measurement-readiness/SKILL.md)                       | Conversion measurement checks before optimising or testing
                    | [test-design-and-power](test-design-and-power/SKILL.md)                       | Test design, minimum detectable effect, sample size, duration
                    | [lift-and-geo-experiments](lift-and-geo-experiments/SKILL.md)                 | Campaign experiments, Conversion Lift, geo experiments
                    | [causal-impact-analysis](causal-impact-analysis/SKILL.md)                     | Pre/post analysis with open-source CausalImpact
                    | [mmm-meridian](mmm-meridian/SKILL.md)                                         | Optional: MMM with open-source Meridian
Planning            | [account-structure-plan](account-structure-plan/SKILL.md)                     | Consolidated structure and migration plan
                    | [budget-seasonality-plan](budget-seasonality-plan/SKILL.md)                   | Monthly and peak budgets, seasonality
                    | [market-expansion-plan](market-expansion-plan/SKILL.md)                       | New-country launch plan

## Related public skill

The diagnostic skills extend Google's official
[`google-ads-api-account-diagnostics`](http://google3/third_party/skills/skills/ads/google-ads-api-account-diagnostics/SKILL.md)
skill, which is published at github.com/google/skills. Each skill still works
without it.

## Before using with an advertiser

-   [ ] Replace **G** rules with public sources, or relabel them **W**; confirm
    links for **P** rules.
-   [ ] Agree the **W** bands with the advertiser, and have them fill in
    `client_rules.md`.
-   [ ] Run each skill's `evals/cases.md` on the advertiser's agent harness. All
    safety cases must pass.
-   [ ] Test the field combinations the skills mark "check with
    `get_resource_metadata`" on a live account.
