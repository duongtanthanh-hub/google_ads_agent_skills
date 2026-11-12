load(
    "//learning/gemini/agents/customization_rules:defs.bzl",
    "agent_skill",
    "agent_skill_test",
)

package(default_visibility = ["//visibility:public"])

agent_skill(
    name = "shopping-performance-diagnostic",
    srcs = [
        "SKILL.md",
        "evals/cases.md",
        "references/01_measurement.md",
        "references/02_delivery_impression_share.md",
        "references/03_bids_caps_priority.md",
        "references/04_recent_changes.md",
        "references/05_products.md",
        "references/06_overlap.md",
        "references/07_lag_seasonality.md",
        "references/queries.md",
    ],
)

agent_skill_test(
    name = "shopping-performance-diagnostic_test",
    agent_skill = ":shopping-performance-diagnostic",
)
