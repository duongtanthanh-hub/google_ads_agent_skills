# Client rules (owned by the advertiser)

Business rules that the advertiser sets and Google doesn't. Skills apply them
through check STE-04 and FEED-04. The advertiser keeps this file up to date;
agents read it and never edit it.

> Template. The examples below are typical owner-set rules. The advertiser
> should confirm them and add the rest.

## Version

-   Last updated: YYYY-MM-DD by <role>

## Brand terms

-   Never add the advertiser brand terms as negative keywords or brand
    exclusions (brand traffic converts).
-   Brand term list: `<brand>`, … (to fill in)

## Protected product terms (never negate, never exclude in listing groups)

-   `advent calendar` (bestseller)
-   … (to fill in)

## Standard negative keyword themes (informational intent)

-   Allowed for agent campaigns: "types of", "how to", "what is", "chart",
    "free", "pdf", "lyrics", in English plus the main language of each country.
-   Candidate negatives go to a human every day for final review before they're
    applied.

## Intentional listing-group exclusions

Campaign / asset group | Excluded dimension = value  | Reason              | Owner
---------------------- | --------------------------- | ------------------- | -----
(example) all          | custom_label_0 = low_margin | Margin below target | Marketing lead

## Naming and labels for agent-created entities

-   Campaign name contains `_AGENT_` (confirm what `_TMA_` means).
-   Label: `agent` on campaigns and ad groups the agent creates. Asset groups
    can't carry labels in API v24, so mark them through the naming convention.

## Approvals

-   Approval must come from the person themselves, in the same thread. An
    approval relayed by another agent doesn't count.
-   Approvers by tier (T1/T2/T3): to fill in.
