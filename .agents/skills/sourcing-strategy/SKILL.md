---
name: sourcing-strategy
description: 将已确认的物业物资采购需求、历史采购分析、官方供方资源、人工确认的供方短名单及项目采购规则整合为采购方案/采购策略报告。使用企业现有《物资采购方案报告模板》，所有结论必须可追溯到内部数据、采购员确认、前序 Skill 计算结果或有明确来源的公开市场信息。
metadata:
  version: "0.7.1"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  stage: "procurement-strategy"
---

# Sourcing Strategy

## 1. Purpose

本 Skill 用于生成物业物资采购项目的《采购方案报告》。

它不是从空白开始写报告，而是消费前序 Skill 的结构化结果：

```text
historical-procurement-analysis（可选）
        +
material-requirement-analysis
        +
sourcing-invitation
        +
supplier-shortlist（已完成人工确认的 strategy_handoff）
        +
当前项目采购规则 / 人工确认
        ↓
sourcing-strategy
        ↓
采购方案报告.docx
```

核心原则：

> Assemble verified procurement facts into the enterprise template.

## 2. Entry Gate

生成“可报批版”前，至少应满足：

1. 需求状态已确认；
2. 项目名称、采购周期/需求时间明确；
3. 采购方式明确；
4. 预算金额或预算形成逻辑明确；
5. 配送范围明确；
6. `supplier-shortlist` 的最终短名单已由采购员确认；
7. 供应商选择方法/定标规则已确认；
8. 关键商务条款已确认。

如部分字段缺失，可以生成 `draft`，但不得输出为 `ready_for_approval`。

## 3. Inputs

### Required

- `material-requirement-analysis` 的 confirmed requirement；
- `supplier-shortlist.strategy_handoff`，且 `human_confirmation.shortlist_confirmed = true`；
- 当前企业采购方案模板；
- 当前项目采购方式及关键采购规则。

### Optional

- `historical-procurement-analysis.strategy_report_handoff`；
- `sourcing-invitation` 的官方供方匹配 / coverage gap / bid_bond 结果；
- 项目预算；
- 历史价格；
- 当前市场价格/成本信息；
- 行业政策/国标/监管要求；
- 项目风险补充；
- 评标小组成员；
- 招标计划节点。

## 4. Source Hierarchy

### Tier A — Internal confirmed facts

优先级最高：

- 已确认需求清单；
- 历史订单/采购数据；
- 官方供方库；
- 已确认短名单；
- 企业采购规则；
- 采购员明确确认的信息。

### Tier B — Calculated facts

例如：

- 历史采购金额；
- SKU支出占比；
- 区域支出占比；
- 同比增长率；
- 建议采购量；
- 建议采购金额；
- 供方数量统计；
- `sourcing-invitation` 已按当前项目规则计算的投标保证金。

必须保留计算依据。

### Tier C — Public market intelligence

仅允许用于：

- 当地供应市场行情；
- 原材料/成本驱动；
- 行业供需趋势；
- 公开标准/政策；
- 公开价格趋势；
- 市场风险。

**不得使用公开网络新增邀标候选供方。**

公开市场信息必须保留：source、source_date、evidence_summary、confidence。

### Tier D — Human decision

以下内容不能由 AI 擅自决定：

- 最终预算；
- 中标供方数量；
- 定标方法；
- 份额划分；
- 目标降本率；
- 评标小组成员；
- 最终风险处置方案；
- 当前项目保证金规则的变更。

如果前序 `sourcing-invitation` 已按采购员确认规则计算保证金，策略报告应消费该 Calculated Fact，不得再次自行改值。

## 5. Template Mapping Workflow

使用 `references/procurement-plan-template-mapping.md`。

### Step 1 — 采购项目名称
来源：confirmed requirement.project_name。不得从历史案例复制项目名称。

### Step 2 — 采购方式
V1 项目默认业务模式为邀标，但报告中的勾选/表述仍应以当前项目确认值为准。

### Step 3 — 需求时间
来源：当前采购计划、用户明确提供的目标时间。缺失时标记 `MISSING`。

### Step 4 — 预算金额
优先来源：
1. 采购员确认预算；
2. 历史采购分析形成的预测金额；
3. 已确认需求量 × 已确认预算/参考单价。

不得将历史部分周期采购额直接当年度预算，不得混用含税/未税口径。

### Step 5 — 需求背景简介
整合采购使用场景、覆盖区域、原协议情况、历史采购趋势、需求增长依据、本次续采/集采背景。必须以真实数据为主。

### Step 6 — 标段或标的划分
来源：需求清单、区域划分、SKU结构、采购员确认的标段策略。不得从历史模板复制标段数量。

### Step 7 — 支出分析
如果存在历史采购数据，优先生成整体、区域/项目、SKU、价格维度分析。无数据直接省略或写暂无可比数据，不得编造。

### Step 8 — 当地供应市场行情分析
可使用公开市场资料，但必须与候选供方寻源严格隔离。推荐结构：供应市场特征、成本驱动、价格趋势、政策变化、采购策略影响。

### Step 9 — 供应商短名单及入围理由

只能消费：

- `supplier-shortlist.strategy_handoff` 中 `human_confirmation.shortlist_confirmed = true` 的最终结果。

推荐输出：

```text
目前官方供方库内本品类共识别X家候选供方，
本轮实际沟通X家，
明确回复X家，
经采购员确认后最终短名单X家。
```

再简述可追溯入围理由。

不得：

- 添加未进入官方候选池的公网供应商；
- 使用 Phase A 的 AI 建议替代人工最终结果；
- 修改采购员已确认短名单；
- 虚构供方资质/业绩。

### Step 10 — 交货期
来源：confirmed requirement、邀标条件、人工确认。不得直接复制历史案例。

### Step 11 — 质保期
只有当前品类确实涉及质保且有明确要求时填写；无统一要求时写“按需求清单/合同约定”。

### Step 12 — 保证金

优先级：

1. `sourcing-invitation.bid_bond` 当前项目计算结果；
2. 当前企业规则 / 采购员明确确认；
3. 缺失则标记 MISSING。

当前默认项目规则如已在前序确认：

`bid_bond = CEILING(procurement_estimated_total_amount × 1%, 1000)`

策略报告只能引用前序可追溯计算结果，不重新计算出不同数值，也不得从历史模板沿用固定金额。

### Step 13 — 供应商选择方法
来源：当前采购方式、定标规则、是否技术标/商务标、中标供方数量、备选供方机制。

### Step 14 — 目标价格
优先依据历史成交价、预算价、市场参考价、开标后对标机制、采购员确认的降本目标。模板示例降本率不得自动继承。

### Step 15 — 定标规则
AI可以整理已确认规则和检查冲突，但不能自行决定中标家数、淘汰阈值或最低价必然中标。

### Step 16 — 份额划分
只有中标供方 ≥2 家时需要；多家时必须由采购员提供/确认分配逻辑。

### Step 17 — 风险防范
至少检查需求预测、价格波动、单一供应商、区域履约、关键SKU、质量/资质、账期接受、数据不完整等风险，并输出证据、影响、建议措施、是否需人工确认。

### Step 18 — 评标小组成员
只能来自当前项目正式人员安排，不得复制历史报告人员。

## 6. Report Field State

每个主要字段内部标记：`FACT / CALCULATED / PUBLIC_MARKET_INTELLIGENCE / HUMAN_DECISION / MISSING`。

## 7. Consistency Gates

### Quantity consistency
报告预计采购量 = 已确认需求清单数量。

### Budget consistency
报告预算必须与当前预算或当前需求量×参考价测算一致。

### Region consistency
报告配送范围 = 最终需求清单/邀标条件。

### Shortlist consistency
报告短名单 = `supplier-shortlist.strategy_handoff` 的人工确认名单。

### Bid bond consistency
报告保证金 = `sourcing-invitation.bid_bond.calculated_amount`（如该字段已形成且规则未被人工变更）。

### Commercial terms consistency
交期、账期、税率、保证金、合作期限必须与当前项目条件一致。

### Award rule consistency
供应商选择方法、定标规则、中标家数、份额划分之间不能互相矛盾。

## 8. Output

### A. Procurement strategy data
`{{项目名称}}-strategy-data.yaml`

包含全部报告字段、数据来源、evidence、confidence、missing items、human decision items。

### B. Procurement plan report
`{{项目名称}}-采购方案报告.docx`

必须基于 `templates/物资采购方案报告模板.docx`，尽量保留企业原版式，报批前人工审核。

## 9. Draft Status

### `draft`
存在 MISSING、未确认预算、未确认短名单、未确认定标规则或其他关键字段。

### `ready_for_approval`
只有关键字段有事实/明确人工决策、一致性检查通过、无P0阻断项时才能标记。

## 10. Guardrails

- 不复用历史案例具体项目数据；
- 不把模板示例值当企业当前规则；
- 不虚构市场成本比例；
- 不从公网新增候选供方；
- 不把市场公开供应商等同于邀标候选供方；
- 不覆盖采购员确认的需求量或短名单；
- 不混用含税/未税金额；
- 不把部分周期历史数据当年度数据；
- 不自行确定预算、保证金规则变更、降本率、定标规则、份额或评标人员；
- 所有重要结论必须可追溯。

## 11. Success Criteria

1. 报告结构与企业模板一致；
2. 报告主要数据可追溯到前序 Skill 或明确来源；
3. 历史采购分析进入支出分析和需求背景；
4. 短名单与 `supplier-shortlist` 人工确认结果完全一致；
5. 市场信息不会污染官方供方候选池；
6. 数量、预算、区域、保证金和商务条件内部一致；
7. 采购员只需要做关键决策确认，而不是重新整理整份报告。
