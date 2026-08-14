# CHANGELOG

## v0.5.0

新增：

- `historical-procurement-analysis/SKILL.md`
- `historical-procurement.schema.yaml`
- `historical-procurement-analysis/references/forecast-rules.md`
- 历史采购分析结构化 Handoff 示例

流程调整：

- 在 `material-requirement-analysis` 前新增“历史采购数据（可选）”前置环节。
- 有历史采购数据时，先分析历史采购量、金额、单价及采购规模变化，再形成需求数量/金额基线。
- 无历史采购数据时直接跳过，不影响现有需求分析流程。
- 需求清单的预计采购量可使用上一完整年度实际采购量，并在有明确业务/采购规模增长依据时按比例测算。
- 数量增长和价格增长必须拆分，禁止将同一增长率重复计算到预计金额。
- 新增 `strategy_report_handoff`，为后续采购策略报告提供支出分析、历史趋势、测算依据、异常与风险等事实数据。

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
