# CHANGELOG

## v0.6.0

新增：

- `shortlist-approval/SKILL.md`
- `shortlist-approval.schema.yaml`
- `shortlist-approval/templates/shortlist-approval-email.md`
- 短名单报批 Handoff 示例

流程调整：

- `supplier-shortlist` 输出经采购员确认后进入 `shortlist-approval`。
- 本阶段不重新评分、不重新筛选供方，只整理已确认短名单。
- 固定输出“短名单报批邮件草稿 + strategy_handoff”。
- 入围/未入围原因必须来自短名单表、供方真实回复或官方数据。
- 未经采购员确认不得将“建议入围”写成“已入围”。
- 邮件仅生成草稿，不自动发送。

## v0.5.2

基于米面粮油真实历史订单与现有需求清单联合验证：

- 新增 `historical-procurement-analysis/references/region-quantity-mapping-rules.md`；
- 历史采购量聚合从单纯 SKU 扩展为 `SKU + 区域`；
- 新增重复需求行 Gate：同名同规格重复行如果没有显式区域/项目字段，禁止自动回填数量；
- 区域识别优先使用订单导出的 `地址省份 / 地址城市 / 收货地址 / 使用部门`；
- 部分周期历史数据只作为 observed baseline，不允许按天数/月数机械年化；
- 区域采购结构、SKU支出结构进入后续采购策略报告 Handoff。

## v0.5.1

基于真实订单导出样本验证：

- 新增 `historical-procurement-analysis/references/order-export-field-mapping.md`；
- 固定订单导出字段与历史采购标准字段的映射；
- 新增周期完整性 Gate：部分月份/周度数据可汇总，但不得默认年化；
- SKU 聚合优先使用 `物料编码 + 规格 + 单位`；
- 增加 `数量 × 单价 ≈ 行级金额` 的一致性检查；
- 明确真实订单联系人、电话、地址、供应商名称等数据不上传公开/代码示例，仅保留脱敏规则。

## v0.5.0

新增：

- `historical-procurement-analysis/SKILL.md`
- `historical-procurement.schema.yaml`
- `historical-procurement-analysis/references/forecast-rules.md`
- 历史采购预测 Handoff 示例

流程调整：

- 历史采购分析作为可选前置步骤；无历史数据项目可直接进入需求分析。
- 历史实际、AI计算、假设和人工确认必须分开记录。
- 采购规模增长与价格增长分开计算，禁止重复放大金额。
- 预测数量只有在历史数量可比且存在明确增长依据时才允许回填需求清单。
- 历史分析结果同步形成 `strategy_report_handoff`，供后续采购策略报告使用。

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
