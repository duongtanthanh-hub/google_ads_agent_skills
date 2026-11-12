load(
    "//learning/gemini/agents/customization_rules:defs.bzl",
    "agent_skill",
    "agent_skill_test",
)

package(default_visibility = ["//visibility:public"])

agent_skill(
    name = "ads-change-guardrails",
    srcs = [
        "SKILL.md",
        "evals/cases.md",
        "references/01_change_types.md",
        "references/02_approval_tiers.md",
        "references/03_options_templates.md",
        "references/queries.md",
    ],
)

agent_skill_test(
    name = "ads-change-guardrails_test",
    agent_skill = ":ads-change-guardrails",
)
