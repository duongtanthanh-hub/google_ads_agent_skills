# 3. Creative (per ad)

Demand Gen ad types in the API (`ad_group_ad.ad.type`):
`DEMAND_GEN_MULTI_ASSET_AD` (images), `DEMAND_GEN_CAROUSEL_AD` (2–10 cards),
`DEMAND_GEN_VIDEO_RESPONSIVE_AD` (video), `DEMAND_GEN_PRODUCT_AD` (feed). Count
only ads with `ad_group_ad.status != 'REMOVED'`, and only assets that are
approved or under review.

Check  | What to verify                                | Data                                                                                                                                                                                                                                                       | How to judge
------ | --------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------
DG-05  | Image coverage (multi-asset and carousel ads) | `ad_group_ad.ad.demand_gen_multi_asset_ad.marketing_images` (1.91:1), `.square_marketing_images` (1:1), `.portrait_marketing_images` (4:5), `.tall_portrait_marketing_images` (9:16); image size from `asset.image_asset.full_size.*`                      | PASS: 1.91:1, 1:1 and 4:5 all present (9:16 recommended) and ≥ 5 images in total (maximum 20 across the four lists). WARN: one of those three ratios missing, or < 5 images. BLOCK: neither landscape nor square
DG-06  | Video coverage                                | `ad_group_ad.ad.demand_gen_video_responsive_ad.videos` → `asset.orientation` (`LANDSCAPE`, `PORTRAIT`, `SQUARE`)                                                                                                                                           | PASS: ≥ 1 landscape (16:9) and ≥ 1 vertical (9:16) video in each ad group that can serve on YouTube; square (1:1) recommended. WARN: no vertical video while Shorts is enabled, or no video at all while YouTube channels are on
DG-07  | Text and brand assets                         | Multi-asset: `.headlines` (≤ 40 characters, max 5; keep at least one ≤ 30), `.descriptions` (≤ 90, max 5), `.business_name` (≤ 25), `.logo_images` (1:1, max 5). Video: `.headlines`, `.long_headlines`, `.descriptions`, `.business_name`, `.logo_images` | PASS: 5 headlines and 5 descriptions. WARN: 1–4 of either. BLOCK: no business name or no square logo
AST-08 | Policy status of ads and assets               | `ad_group_ad.policy_summary.approval_status`; `ad_group_ad.primary_status`                                                                                                                                                                                 | BLOCK if an ad in an ad group about to be enabled is `DISAPPROVED`; WARN for `APPROVED_LIMITED`

**Ad strength.** Read `ad_group_ad.ad_strength` if it's returned for Demand Gen
ads and report it as context. Don't block on it: DG-05…07 are the measurable
checks.

Character limits, maximums and ratios above come from Google Ads Help (Demand
Gen image and video ad specifications, P). Google changes them from time to
time; if `validate_only` disagrees, trust the API error and report it.

**Image and video quality, beyond the counts**

-   Lifestyle images (products in use, gifting moments) work better on Discover
    and Gmail than plain catalogue shots. Keep product images clean, without
    price badges or heavy text overlays.
-   Keep key content in the centre of 4:5 and 9:16 assets; the interface covers
    the top and bottom.
-   Videos under 10 seconds can't serve in YouTube In-Stream; keep at least one
    longer video if In-Stream matters.
-   Each ad group should tell one story (e.g. "personalised Christmas
    ornaments"), not mix unrelated product lines.

**Automatic enhancements.** `ad_group_ad.ad_group_ad_asset_automation_settings`
shows opt-ins such as `GENERATE_VERTICAL_YOUTUBE_VIDEOS`,
`GENERATE_SHORTER_YOUTUBE_VIDEOS`, `GENERATE_DESIGN_VERSIONS_FOR_IMAGES` and
`GENERATE_VIDEOS_FROM_OTHER_ASSETS`. They help fill gaps, but don't count
generated assets towards DG-05 or DG-06; recommend real assets instead.

**Common pattern**

-   Designs change with seasons. Plan a creative refresh before peak dates;
    replace assets gradually rather than all at once (a full swap resets what
    the system has learned about the ad).
-   Use the market's language in every text asset; don't mix English text with a
    localised landing page.
