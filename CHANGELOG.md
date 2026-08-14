# CHANGELOG

## v0.4.0

新增：

- `supplier-shortlist/SKILL.md`
- 企业现有《供方短名单》版式的脱敏空白模板

流程调整：

- 供方回复阶段简化为：`供方回复 → 短名单汇总表`。
- 不在该阶段执行复杂评分、报价排名或回标模型。
- 短名单表仅整理“建议入围 / 待澄清 / 不建议入围”及对应真实原因，最终由采购员确认。
- 保留官方库长名单中参与本轮沟通的全部供方，包括未入围供方，便于过程留痕。

## v0.3.0

新增：

- `sourcing-invitation/SKILL.md`
- `sourcing-invitation-package.schema.yaml`
- 邀标邮件变量模板
- 《招标意向征集登记表》附件模板
- 邀标三件套一致性检查规则
- 米面粮油邀标包黄金样例

流程调整：

- 取消独立 `supplier-intention-screening` Skill。
- `official-supplier-matching` 经采购员确认后直接进入 `sourcing-invitation`。
- `sourcing-invitation` 固定输出：邀标邮件正文 + 最终需求清单 + 招标意向征集登记表。

## v0.2.0

新增：

- `official-supplier-matching/SKILL.md`
- `supplier-match.schema.yaml`
- `official-supplier-source-policy.md`
- `category-and-region-matching-rules.md`
- 米面粮油官方供方库匹配黄金案例

调整：

- 明确官方供方库 ONLY。
- 明确 `in registry ≠ active`。
- 明确跨区域供货能力不得由 AI 推断。
- 明确临时供应商记录不能自动等价为正式邀标供方。

## v0.1.0

新增：

- `material-requirement-analysis/SKILL.md`
- `requirement.schema.yaml`
- `supplier.schema.yaml`
- 米面粮油需求诊断黄金案例
