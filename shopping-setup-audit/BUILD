load(
    "//learning/gemini/agents/customization_rules:defs.bzl",
    "agent_skill",
    "agent_skill_test",
)

package(default_visibility = ["//visibility:public"])

agent_skill(
    name = "shopping-setup-audit",
    srcs = [
        "SKILL.md",
        "evals/cases.md",
        "references/01_measurement.md",
        "references/02_bidding_budget.md",
        "references/03_structure_priority.md",
        "references/04_products_feed.md",
        "references/05_negatives.md",
        "references/06_launch_safety.md",
        "references/queries.md",
    ],
)

agent_skill_test(
    name = "shopping-setup-audit_test",
    agent_skill = ":shopping-setup-audit",
)
