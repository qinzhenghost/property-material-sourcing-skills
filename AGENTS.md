# Repository Agent Instructions

## Primary entrypoint

For end-to-end物业物资采购、邀标寻源、需求分析、供方筛选、短名单、采购策略/采购方案报告任务，优先使用：

`.agents/skills/property-material-sourcing/SKILL.md`

Do not require the user to manually choose among the five internal modules.

## Internal modules

The following Skills are implementation modules orchestrated by the primary entrypoint:

- `historical-procurement-analysis`
- `material-requirement-analysis`
- `sourcing-invitation`
- `supplier-shortlist`
- `sourcing-strategy`

They may be invoked directly only when the user explicitly requests a single stage, debugging, regression testing, or maintenance of that module.

## Resume behavior

Always continue from the latest verified human-confirmed checkpoint. Do not restart completed phases unless upstream facts changed.

## Human gates

Never bypass procurement human confirmation for agreement scope, unresolved P0 requirements, initial invitation supplier pool, final shortlist, formal award/budget/share/target decisions, or final approval.

## Supplier source guardrail

Invitation candidate supplier identity must originate from the enterprise official supplier registry. Public web research may support market intelligence in the strategy phase but must never add invitation candidates.

## Template guardrail

Use the validated Office templates stored under the internal module template directories. Do not rebuild or move binary templates merely because the workflow is orchestrated by one primary Skill.

## Canonical path

`.agents/skills/` is canonical. `skills/` is a compatibility mirror until explicitly retired.
