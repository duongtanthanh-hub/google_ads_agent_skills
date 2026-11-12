# 0. Pre-flight, account selection and fallback

A short, self-contained version of the steps in the public
`google-ads-api-account-diagnostics` skill (with `google-ads-api-mcp-setup` for
installation and `google-ads-api-quickstart` for API basics). Follow it even if
those skills aren't loaded.

## 0.1 Check the tools

The read-only Google Ads MCP server exposes `search` (GAQL; arguments
`customer_id` and `query`), `list_accessible_customers` and
`get_resource_metadata`. Names may carry a server prefix (for example
`customers_list_accessible_customers`).

-   **Tools present:** continue with 0.2.
-   **Tools missing:** go to 0.4. Don't fall back to any write tool for reads.

## 0.2 Pick the account

If the customer ID isn't given:

1.  Call `list_accessible_customers`.
2.  Under each accessible customer, query `customer_client` and keep enabled,
    non-manager accounts only (Q0 in [queries.md](queries.md)).
3.  Run the diagnostic only on those IDs. Querying manager or cancelled accounts
    returns errors.

If several client accounts run Demand Gen, diagnose the one(s) named in the
question; otherwise rank by Demand Gen cost in the last 30 days and say which
you chose.

## 0.3 Check field names first

Run `get_resource_metadata` for `campaign`, `ad_group`, `ad_group_ad` and
`ad_group_ad_asset_view` before the first query. Demand Gen fields change
between API versions (for example, the `maps` channel control is new). If a
field in [queries.md](queries.md) isn't listed or isn't `selectable_with` the
segment you need, drop it and mark the dependent check NOT_ASSESSED. Don't retry
the same failing query in a loop.

Quota: these reads are light, but don't schedule hourly full pulls; use
`change_status` / `change_event` for incremental monitoring (EXE-07). On
`RESOURCE_EXHAUSTED`, back off and report it (EXE-06); never fail silently.

## 0.4 Fallback when the tools are unavailable

1.  Say that live diagnostics need the Google Ads MCP server and point to the
    `google-ads-api-mcp-setup` skill for setup.
2.  Output the full plan: the windows you would use, every GAQL query from
    [queries.md](queries.md) with placeholders filled where known, and what each
    result would mean (the "How to judge" columns in the checklists).
3.  Include the gotchas: `metrics.cost_micros / 1,000,000` for currency;
    `change_event` needs a date filter within the last 30 days and
    `LIMIT ≤ 10000`, and can't select metrics; attribution-type and
    conversion-action segments can't be combined with cost.
4.  **Don't** run shell commands, local scripts, database queries or RPC tools
    to get account data, and don't use another agent's credentials. You're
    blocked until the tools exist; say so.
