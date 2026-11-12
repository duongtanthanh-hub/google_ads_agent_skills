# 5. Negatives and client rules

Standard Shopping has no keywords, so negative keywords and campaign priority
are the only query-level controls. They are also where agents most easily cut
off traffic that converts.

Check  | What to verify                                     | Data                                                                                                                                                    | How to judge
------ | -------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
STE-04 | Negatives and brand exclusions follow client rules | Campaign and ad group negatives, shared negative lists and brand lists (Q8a–Q8c); `client_rules.md`                                                     | BLOCK if any own-brand term is negated or excluded, or any protected product term is negated (any match type, any language). PASS otherwise
SHP-05 | Each new negative is backed by evidence            | Search terms for the campaign (Q9) over the SHP-05 window, with clicks, cost, conversions and conversion value; order data for the term where available | PASS: every candidate meets the SHP-05 evidence rule. WARN: shorter evidence window, or the term converted but a human reviewed it and the reason is recorded. BLOCK: the term has conversions or orders and no human has reviewed it

**How to check a candidate negative**

1.  Match it against the brand and protected-term lists in `client_rules.md` as
    a substring, in every match type, including translations. A phrase or broad
    negative such as "ideas" can block a protected term ("advent calendar
    ideas").
2.  Pull the search terms it would block from Q9 (exact, phrase and broad
    matches of the negative) and total their conversions and value.
3.  Check the informational themes allowed in `client_rules.md` ("how to",
    "free", "pdf", …). Anything outside those themes goes to a human.
4.  Translate only after checking the meaning in that country's main language.

**Other things to note**

-   Shared negative lists apply to every campaign they are attached to. A new
    term on a shared list has the blast radius of all those campaigns: count
    them for EXE-05.
-   Brand lists can apply to Shopping;
    `campaign.shopping_setting.ignore_brand_exclusion_in_shopping_ads` (v24)
    shows whether brand exclusions are ignored for Shopping ads in that
    campaign. Own-brand terms must never be excluded either way.
-   Log every negative added (who approved, when, which campaigns or lists) so
    it can be reviewed and rolled back.
