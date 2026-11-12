# Google Ads agent skills: change safety, Shopping and Demand Gen

This package contains **7 Agent Skills** for AI agents that manage Google Ads accounts, plus the shared rule files they depend on. They help agents **set up Shopping and Demand Gen campaigns well, make changes safely, and diagnose results**, before and after anything is written to Google Ads.

The skills use the `SKILL.md` format (YAML front matter + Markdown, with `references/` and `evals/` folders), which Claude Code and similar agent harnesses load directly. They read account data through the Google Ads MCP server (`search`, `list_accessible_customers`, `get_resource_metadata`) and use Google Ads API v24 field names.

> [!IMPORTANT]
> **Draft / pilot version.** These skills are best-practice recommendations, not official Google policy. Every rule has a source label (see "Rule labels" below). Review the rules marked **W** before relying on them, and run the eval cases on your own setup first.

## What's included

| Skill | What it does | Typical user |
| --- | --- | --- |
| [ads-change-guardrails](ads-change-guardrails/SKILL.md) | Checks **one proposed change** (budget, target, bid strategy, pause, structure, negatives) before approval. Gives the approval tier and, when a request breaks a rule, offers a staged alternative instead of refusing | Planning agent, reviewer agent |
| [ads-bulk-change-protocol](ads-bulk-change-protocol/SKILL.md) | Plans **large jobs** (> 100 entities or > 10,000 operations): exact-payload dry run, batch jobs vs chunked requests, waves, failure-rate caps, reconciling failed operations, rollback | Planning agent, reviewer agent |
| [ads-post-change-check](ads-post-change-check/SKILL.md) | Checks results **after** a change at D+1, D+3, D+7 and D+14 against a baseline; rollback triggers | Analytics agent |
| [shopping-setup-audit](shopping-setup-audit/SKILL.md) | Audits a **Standard Shopping** setup before and after launch: bidding and CPC caps, budget, priority, product groups, overlap with Performance Max, launch size | Planning agent, reviewer agent |
| [shopping-performance-diagnostic](shopping-performance-diagnostic/SKILL.md) | Explains **why Shopping results changed**: measurement, impression share, bid caps, recent changes, product performance | Analytics agent |
| [demandgen-setup-audit](demandgen-setup-audit/SKILL.md) | Audits a **Demand Gen** setup: bidding vs goal, budget sizing, channels, creative (incl. vertical video), audiences, product feed, measurement plan | Planning agent, reviewer agent |
| [demandgen-performance-diagnostic](demandgen-performance-diagnostic/SKILL.md) | Explains **why Demand Gen results changed**. Reports click / engaged-view / view-through conversions separately, and routes "is Demand Gen working?" questions to a proper test | Analytics agent |

**Shared files (required; keep them next to the skill folders):**
- [_shared/thresholds.md](_shared/thresholds.md): every number the skills use, as rule IDs such as `CHG-01`.
- [_shared/output_contract.md](_shared/output_contract.md): the common answer format: PASS / WARN / BLOCK verdict, approval tier, findings with evidence, and a JSON block for your orchestration layer or audit trail.
- [_shared/client_rules.md](_shared/client_rules.md): **the advertiser's own rules.** A template to complete (see Setup).

## How they fit an agent workflow

| Step | Who | Skill |
| --- | --- | --- |
| 1. Plan a change | The agent that proposes and executes Google Ads changes | Setup audit for the campaign type, plus `ads-change-guardrails`, or `ads-bulk-change-protocol` for large jobs |
| 2. Review | A separate reviewer agent (recommended, ideally on a different model) | Re-runs the same skills independently; confirms the approval tier |
| 3. Approve | The right person, by tier | The approval comes from that person directly, in the same thread. An approval relayed by another agent never counts |
| 4. Execute | The executing agent | Dry run first, then the write (or wave 1 for bulk jobs) |
| 5. Check | The analytics agent | `ads-post-change-check` at D+1/3/7/14; a diagnostic skill if results move |

**Approval tiers** (defined in `thresholds.md`, EXE-05):

| Tier | When | Who approves |
| --- | --- | --- |
| T1 | Small, within the change limits | Reviewer agent |
| T2 | Larger, or a warning on change size or spacing | Team lead |
| T3 | Bulk, cross-account, many new campaigns, or overriding a BLOCK | Admin, with a staged rollout |

## Install
1. Copy the whole package folder into your agent's skills directory (for Claude Code, e.g. `.claude/skills/`). **Keep `_shared/` alongside the skill folders,** because every skill reads `../_shared/`.
2. Make sure the agents that use the skills can reach the Google Ads MCP server.
3. Optional but recommended: also install Google's official public skills from github.com/google/skills: `google-ads-api-account-diagnostics`, `google-ads-api-mcp-setup` and `google-ads-api-quickstart`. These skills work without them.

## Setup (before first use)
1. **Complete `_shared/client_rules.md`:**
   - brand terms (never added as negatives);
   - protected products and terms;
   - intentional product exclusions;
   - naming and labels for agent-created campaigns (labels go on campaigns and ad groups; asset groups use naming only);
   - approvers for T1/T2/T3;
   - break-even ROAS per market.
2. **Review and agree the W rules.** The main ones for this package:

   | Rules | What they cover |
   | --- | --- |
   | CHG-01…06 | Budget, target and CPC-cap steps (≤ 20%, ≥ 3 days apart); no pausing or restructuring in the first 14 days |
   | STR-01/02 | Enough conversions per campaign; launch batch size |
   | EXE-03/04 | Bulk-job failure cap (halt above 2%); waves |
   | SHP / DG rules | Shopping and Demand Gen specifics |

3. **Decide the source of truth for revenue** (Google Ads conversion value vs your data warehouse vs GA4) and record it in `client_rules.md`.

## Suggested rollout (4–6 weeks)

| Week | What |
| --- | --- |
| 1 | Install. Run the eval cases (below). Run `shopping-setup-audit` and `demandgen-setup-audit` in **live** mode on current campaigns, read-only, to see the findings |
| 2–5 | Every agent change goes through `ads-change-guardrails` (and `ads-bulk-change-protocol` for big jobs), then `ads-post-change-check`. Weekly diagnostics on Shopping and Demand Gen |
| 6 | Review: how often agents were blocked or warned, false alarms, approval wait times, and results of changed campaigns vs before |

## Eval cases
Each skill has `evals/cases.md` with **12 test scenarios**:
- the expected verdict and approval tier;
- the rule IDs it must flag;
- what it must NOT do.

They include **safety cases**, such as brand terms as negatives, tripling budgets, pausing after 3 days, and bulk writes with no dry run. They also include a **clean case**, to catch false alarms.

**Before agents rely on a skill, all its safety cases should pass.** The cases are written as a scoring guide. You can run them by hand, or automate them by giving each scenario to the agent and comparing its verdict, rule IDs and tier with the expected ones.

## Rule labels

| Label | Meaning |
| --- | --- |
| **P** | Based on public Google documentation (Google Ads Help, Merchant Center Help, Google Ads API docs) |
| **G** | Google best-practice guidance; the exact public wording is still being confirmed |
| **W** | Working proposal. Not Google policy; agree it with the advertiser and tune it in use |
| **C** | The advertiser's own rule (in `client_rules.md`) |

## Skills mentioned but not included
These skills point to some skills that aren't part of this package, usually as "for X, use skill Y instead". The most frequent ones:
- `feed-policy-audit`
- `pmax-setup-audit`
- `lift-and-geo-experiments`
- `test-design-and-power`
- `measurement-readiness`

If an agent reaches one of these pointers, it should report that the skill isn't installed and treat that step as out of scope. `feed-policy-audit` (Merchant Center health) and `lift-and-geo-experiments` (incrementality tests, e.g. for Demand Gen) are the most useful additions.

## Good to know
- **Read-only:** the skills never write to Google Ads. The executing agent writes only after approval.
- **Fields to confirm:** some API field combinations are marked "check with `get_resource_metadata`". Confirm them on your accounts during the first week.
- **API access:** Google Ads API access levels belong to the Google Cloud project used for authentication. Quota planning in `ads-bulk-change-protocol` uses that project's access level.
- **Questions:** contact your Google account team.
