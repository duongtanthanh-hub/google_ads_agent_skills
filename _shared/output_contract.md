# Output contract (all audit and review skills)

Every audit skill returns the same structure, so the Ads Reviewer agent, the
Workstation and humans can read any skill's result the same way.

## 1. Verdict line (first line of the reply)

```
VERDICT: BLOCK | WARN | PASS   ·   scope: <account / campaign / plan id>   ·   approval tier: T1 | T2 | T3
```

-   **BLOCK** if any finding is BLOCK.
-   **WARN** if there is no BLOCK and at least one WARN.
-   **PASS** only if every check is PASS. NOT_ASSESSED checks are listed but
    don't count as PASS.
-   The approval tier comes from EXE-05 in `thresholds.md`.

## 2. Short summary (≤ 5 bullets)

The biggest risks first, in plain language, each with the fix.

## 3. Findings table

\#  | Check ID | Scope        | Status | Evidence (actual vs threshold)                    | Recommended action                              | Src
--- | -------- | ------------ | ------ | ------------------------------------------------- | ----------------------------------------------- | ---
1   | BUD-01   | Campaign 123 | WARN   | Budget \$40/day = 1.6× CPA (\$25); threshold ≥ 3× | Raise to ≥ \$75/day, or merge into campaign 456 | P

Rules:

-   One row per failed or not-assessed check per entity. Group the PASS rows
    into one line per section ("Assets: 8/8 PASS").
-   **Evidence** must give the actual value and the threshold. Never write
    "looks low".
-   Recommendations are suggestions ("Suggest…", "Consider…"), but BLOCK rows
    state plainly that the change shouldn't go ahead as planned.

## 4. Machine-readable block (for the Workstation)

```json
{
  "skill": "pmax-setup-audit",
  "version": "0.1",
  "verdict": "WARN",
  "approval_tier": "T2",
  "findings": [
    {"check_id": "BUD-01", "scope": "campaign:123", "status": "WARN",
     "actual": 1.6, "threshold": ">=3x", "unit": "budget/CPA",
     "action": "Raise daily budget to >= 75 USD or merge into campaign 456",
     "source": "P"}
  ],
  "not_assessed": ["MEA-04"],
  "client_rules_version": "2026-11-04"
}
```

## 5. Overrides

If a human approves despite a BLOCK, the Workstation records who approved, the
check ID, the reason and the rollback plan in the audit trail. An agent can
never record an override on behalf of a person.
