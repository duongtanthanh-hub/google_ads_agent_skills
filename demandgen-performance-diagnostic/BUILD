load(
    "//learning/gemini/agents/customization_rules:defs.bzl",
    "agent_skill",
    "agent_skill_test",
)

package(default_visibility = ["//visibility:public"])

agent_skill(
    name = "demandgen-performance-diagnostic",
    srcs = [
        "SKILL.md",
        "evals/cases.md",
        "references/00_preflight.md",
        "references/01_scope_windows.md",
        "references/02_measurement.md",
        "references/03_delivery_bidding.md",
        "references/04_changes_learning.md",
        "references/05_channels_creative_audience.md",
        "references/06_product_feed.md",
        "references/07_incrementality_routing.md",
        "references/queries.md",
    ],
)

agent_skill_test(
    name = "demandgen-performance-diagnostic_test",
    agent_skill = ":demandgen-performance-diagnostic",
)
