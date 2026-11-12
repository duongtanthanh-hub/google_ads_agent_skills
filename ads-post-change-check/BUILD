load(
    "//learning/gemini/agents/customization_rules:defs.bzl",
    "agent_skill",
    "agent_skill_test",
)

package(default_visibility = ["//visibility:public"])

agent_skill(
    name = "ads-post-change-check",
    srcs = [
        "SKILL.md",
        "evals/cases.md",
        "references/01_checkpoints.md",
        "references/02_baseline_and_lag.md",
        "references/03_rollback_and_stop_rules.md",
        "references/queries.md",
    ],
)

agent_skill_test(
    name = "ads-post-change-check_test",
    agent_skill = ":ads-post-change-check",
)
