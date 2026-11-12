load(
    "//learning/gemini/agents/customization_rules:defs.bzl",
    "agent_skill",
    "agent_skill_test",
)

package(default_visibility = ["//visibility:public"])

agent_skill(
    name = "ads-bulk-change-protocol",
    srcs = [
        "SKILL.md",
        "evals/cases.md",
        "references/01_method_and_limits.md",
        "references/02_dry_run_waves_quota.md",
        "references/03_failures_reconciliation_rollback.md",
        "references/queries.md",
    ],
)

agent_skill_test(
    name = "ads-bulk-change-protocol_test",
    agent_skill = ":ads-bulk-change-protocol",
)
