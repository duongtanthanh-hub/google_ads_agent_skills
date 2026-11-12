# 4. Recent changes (DX-04 step 3)

Line up every edit against the daily trend: budgets, caps, priorities, product
groups, statuses.

Check                | What to verify                                                                      | Data                                                                                                                                                                         | How to judge
-------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
Change log           | Edits in the last 30 days on the campaign, its budget, ad groups and product groups | Q8 `change_event` (`change_resource_type` `CAMPAIGN`, `CAMPAIGN_BUDGET`, `AD_GROUP`, `AD_GROUP_CRITERION`; `changed_fields`; `old_resource` / `new_resource`; `client_type`) | List each change with date, field, old → new and source. Look for `target_spend.cpc_bid_ceiling_micros`, `shopping_setting.campaign_priority`, `status`, `amount_micros`, `cpc_bid_micros`, `negative`
CHG-01               | Budget step size and spacing                                                        | Old/new budget and dates                                                                                                                                                     | Bands per CHG-01
Cap / bid steps      | Cap or bid changes and their size                                                   | Old/new `cpc_bid_ceiling_micros` or `cpc_bid_micros`                                                                                                                         | A cap cut followed by a click drop inside the learning window (CHG-03) is a measured cause
Priority change      | Priority edited on an overlapping campaign                                          | `shopping_setting.campaign_priority` in Q8                                                                                                                                   | Traffic moving between campaigns right after = measured cause (DXP-07)
Product-group change | Product groups subdivided, excluded or re-bid                                       | `AD_GROUP_CRITERION` rows with listing-group criteria                                                                                                                        | A new exclusion (`negative = true`) on items that carried value = measured scope cause; check FEED-04
CHG-03 / DX-03       | Time since launch or last structural change                                         | `campaign.start_date_time`, structural edits in Q8                                                                                                                           | Bands per DX-03 and CHG-03; inside the BLOCK band: "too early"
Launch waves         | New overlapping campaigns created in the window                                     | Q8 `resource_change_operation = CREATE` on `CAMPAIGN`; STR-02                                                                                                                | Many same-day launches (e.g. 100+ in one account in a day) explain shifts in existing campaigns via overlap and priority

**How to attribute**

-   Measured only if the break follows the change inside the expected response
    time (CHG-03 learning window for bid-strategy changes; by the D+1 check of
    EXE-08 for status, cap and scope changes) and nothing else changed.
-   Several edits in the same week: report them as one combined cause.
-   "No edits in the 30 days before the change" is a finding; record it.

**Gotchas**

-   `change_event`: filter `change_date_time` within the last 30 days, `LIMIT` ≤
    10,000, no `metrics.*`. For older history use `change_status` (90 days; when
    and what, without old/new values).
-   Merchant Center feed changes and conversion-action edits don't appear here.
-   Don't select `change_event.user_email` unless needed.
