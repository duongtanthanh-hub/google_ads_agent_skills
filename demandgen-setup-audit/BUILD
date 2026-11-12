load(
    "//learning/gemini/agents/customization_rules:defs.bzl",
    "agent_skill",
    "agent_skill_test",
)

package(default_visibility = ["//visibility:public"])

agent_skill(
    name = "demandgen-setup-audit",
    srcs = [
        "SKILL.md",
        "evals/cases.md",
        "references/01_goal_bidding_budget.md",
        "references/02_channels_targeting.md",
        "references/03_creative.md",
        "references/04_product_feed.md",
        "references/05_audiences.md",
        "references/06_measurement.md",
        "references/07_launch_safety.md",
        "references/queries.md",
    ],
)

agent_skill_test(
    name = "demandgen-setup-audit_test",
    agent_skill = ":demandgen-setup-audit",
)
