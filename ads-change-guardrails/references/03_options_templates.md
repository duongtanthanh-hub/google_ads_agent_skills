# 3. Options templates for rule-breaking requests

Use when any check is WARN or BLOCK. The reply always has:

1.  The output-contract block (verdict line, ≤ 5 bullets, findings table, JSON).
2.  **Numbered options**: compliant first (recommended one marked), the original
    request last. Each option states what changes, the check results, the tier
    and the stop rule.
3.  The closing line: "I'll execute option N only after explicit approval from
    <role> in this thread."

Recommendations are suggestions; BLOCK findings are stated plainly. Never
refuse; never present the original as forbidden.

## A. "Triple all budgets" (CHG-01 BLOCK)

```
VERDICT: BLOCK · scope: 10 PMax campaigns (account <id>) · approval tier: T3 as requested (T1 for option 1)

- Tripling is a +200% step; CHG-01 allows ≤ 20% per step (> 50% = BLOCK).
- Expected risk: spend rises at once (a day can reach 2× the new budget); extra
  spend usually buys lower-value traffic, so ROAS often drops before settling.
- 3 of 10 campaigns aren't budget-limited; more budget may change little there.

Options
1. (Recommended) +20% now on the 7 budget-limited campaigns, then another
   step of ≤ 20% each time ≥ 3 days have passed (CHG-01), while the D+3/D+7
   checks pass. Tier T1. Stop rule: hold the ladder at the OPS-07 WARN band;
   propose rollback at its BLOCK band.
2. +50% now on the 7 budget-limited campaigns. CHG-01 WARN. Tier T2.
3. Triple all 10 as requested. CHG-01 BLOCK; carrying it out is an override.
   Tier T3 (admin), with the risk above recorded and D+1/3/7/14 checks.

I'll execute option 1, 2 or 3 only after explicit approval from the approver
for that tier in this thread.
```

## B. "Pause it, it's been 3 days and ROAS is bad" (CHG-03 BLOCK, CHG-04)

-   Finding: 3 days since launch (< 7 = BLOCK); the last 3 days are inside the
    conversion-lag window, so ROAS is still incomplete.
-   Risk of pausing: the decision is based on immature data; restarting later
    starts the Learning status again.
-   Options:
    1.  (Recommended) Keep running; check at D+7 and D+14
        ([ads-post-change-check](../../ads-post-change-check/SKILL.md)); stop
        rule agreed now (e.g. OPS-07 trigger). Tier T1.
    2.  Cut the budget by ≤ 20% to limit exposure while data matures (CHG-01
        PASS if ≥ 3 days since the last budget change). Tier T1.
    3.  Pause now as requested. CHG-03 BLOCK override → T3.
-   Exception: if serving, policy or tracking is broken, OPS-01 applies; say
    which, and pausing becomes a normal option at its own tier.

## C. "Launch one campaign per seller today" (STR-01, STR-02 BLOCK)

-   Options: (1) one campaign per country/goal with asset groups or listing
    groups per seller group (custom label), first wave per EXE-04; (2) a pilot
    within the STR-02 limit for the sellers with the most volume; (3) as
    requested → T3 and the bulk protocol if > 20 campaigns.

## D. "Add '<brand>' as a negative" (STE-04 BLOCK, client rule)

-   Finding: brand term; the client rule says never negate brand terms.
-   Options: (1) don't add it; if the aim is to cut brand cost elsewhere, look
    at brand campaign targets instead; (2) a narrower negative that doesn't
    contain the brand term, if one fits the aim; (3) as requested: this breaks a
    client rule, so it needs a change to `client_rules.md` by the advertiser
    first, not an agent override.

## E. "Switch everything to Maximize clicks with a \$0.05 cap" (BID-01 WARN, CHG-03)

-   Options: (1) keep value-based bidding, adjust the tROAS by ≤ 15% (CHG-02);
    (2) test Maximize clicks in a campaign experiment for ≥ 4 weeks (CHG-05);
    (3) as requested: T2 (WARN), or T3 if any campaign is < 7 days from its last
    structural change.

## Option block in the JSON

Add to the output-contract JSON:

```json
"requested_change": {"type": "budget", "entities": 10, "step_pct": 200},
"options": [
  {"n": 1, "summary": "+20% on 7 budget-limited campaigns, then <=20% steps >=3 days apart",
   "checks": {"CHG-01": "PASS"}, "tier": "T1", "recommended": true,
   "stop_rule": "OPS-07: hold at WARN band, propose rollback at BLOCK band"},
  {"n": 3, "summary": "triple all 10 as requested", "checks": {"CHG-01": "BLOCK"},
   "tier": "T3", "override_of": ["CHG-01"]}
],
"approval": {"status": "pending", "valid_if": "OPS-09"}
```
