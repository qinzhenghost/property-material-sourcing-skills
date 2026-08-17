---
name: sourcing-strategy
description: 将已确认的物业物资采购需求、历史采购分析、官方供方资源、供方短名单及项目采购规则整合为采购方案/采购策略报告。使用企业现有《物资采购方案报告模板》，所有结论必须可追溯到内部数据、采购员确认或有明确来源的公开市场信息。
metadata:
  version: "0.7.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  stage: "procurement-strategy"
---

# Sourcing Strategy

## 1. Purpose

本 Skill 用于生成物业物资采购项目的《采购方案报告》。

它不是从空白开始“写一篇报告”，而是消费前序 Skill 的结构化结果：

```text
historical-procurement-analysis（可选）
        +
material-requirement-analysis
        +
official-supplier-matching
        +
supplier-shortlist
        +
shortlist-approval
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
6. 供方短名单已由采购员确认；
7. 供应商选择方法/定标规则已确认；
8. 关键商务条款已确认。

如部分字段缺失，可以生成 `draft`，但不得输出为 `ready_for_approval`。

## 3. Inputs

### Required

- `material-requirement-analysis` 的 confirmed requirement；
- `shortlist-approval` 的 confirmed shortlist / strategy_handoff；
- 当前企业采购方案模板；
- 当前项目采购方式及关键采购规则。

### Optional

- `historical-procurement-analysis.strategy_report_handoff`；
- `official-supplier-matching` 的官方库覆盖情况；
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
- 供方数量统计。

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

公开市场信息必须保留：

- source;
- source_date;
- evidence_summary;
- confidence.

### Tier D — Human decision

以下内容不能由 AI 擅自决定：

- 最终预算；
- 保证金金额/比例；
- 中标供方数量；
- 定标方法；
- 份额划分；
- 目标降本率；
- 评标小组成员；
- 最终风险处置方案。

## 5. Template Mapping Workflow

使用 `references/procurement-plan-template-mapping.md`。

### Step 1 — 采购项目名称

来源：

- confirmed requirement.project_name

不得从历史案例复制项目名称。

### Step 2 — 采购方式

V1 项目默认业务模式为邀标，但报告中的勾选/表述仍应以当前项目确认值为准。

### Step 3 — 需求时间

来源：

- 当前采购计划；
- 用户明确提供的目标时间。

缺失时标记 `MISSING`。

### Step 4 — 预算金额

优先来源：

1. 采购员确认预算；
2. 历史采购分析形成的预测金额；
3. 已确认需求量 × 已确认预算/参考单价。

不得：

- 将历史部分周期采购额直接当年度预算；
- 将含税/未税口径混用；
- 未说明税口径。

### Step 5 — 需求背景简介

整合：

- 采购使用场景；
- 覆盖项目/区域；
- 原协议情况；
- 历史采购趋势；
- 需求增长依据；
- 本次续采/集采背景。

必须以真实数据为主。

禁止空泛模板话术替代事实。

### Step 6 — 标段或标的划分

来源：

- 需求清单；
- 区域划分；
- SKU结构；
- 采购员确认的标段策略。

如只有单标段：

- 明确写“本项目不划分多个标段”或当前确认方式。

不得因为历史模板存在“2个标段”就自动复制。

### Step 7 — 支出分析

如果存在历史采购数据，优先生成：

#### 整体

- 历史采购额；
- 历史采购周期；
- 预计采购额；
- 同比变化；
- 数据完整性说明。

#### 区域/项目维度

- 区域采购金额；
- 项目分布；
- 区域占比。

#### SKU维度

- Top SKU；
- Top SKU采购金额；
- Top SKU占比；
- 重点谈判SKU。

#### 价格维度

- 历史成交价；
- 历史价格范围；
- 价格变化趋势。

没有数据的维度直接省略或写“暂无可比数据”，不得编造。

### Step 8 — 当地供应市场行情分析

可使用公开市场资料，但必须与候选供方寻源严格隔离。

推荐结构：

1. 供应市场特征；
2. 成本驱动；
3. 价格趋势；
4. 行业/政策变化；
5. 对本次采购策略的影响。

成本构成比例如果无可靠来源：

- 不填写虚假的“原材料X%、人工X%、利润X%”；
- 可使用定性描述；
- 或标记需要采购员/市场调研补充。

### Step 9 — 供应商短名单及入围理由

只能消费：

- `shortlist-approval` 已确认结果。

推荐输出：

```text
目前官方供方库内本品类共识别X家候选供方，
本轮实际沟通X家，
明确回复X家，
经需求及邀标条件确认后，最终短名单X家。
```

再简述入围理由。

不得：

- 添加未进入官方候选池的公网供应商；
- 修改采购员已确认短名单；
- 虚构供方资质/业绩。

### Step 10 — 交货期

来源：

- confirmed requirement；
- 邀标条件；
- 人工确认。

不得直接复制模板中的“X日”或历史案例“7日”。

### Step 11 — 质保期

只有当前品类确实涉及质保且有明确要求时填写。

如无统一要求：

- “按需求清单/合同约定”；
- 不自行设定期限。

### Step 12 — 保证金

保证金属于 Human Decision / Enterprise Rule。

报告中不得因为模板示例存在：

- 预算1%投标保证金；
- 2%履约保证金；

就自动采用。

必须有当前项目明确依据。

### Step 13 — 供应商选择方法

来源：

- 当前采购方式；
- 定标规则；
- 是否技术标；
- 是否100%商务标；
- 中标供方数量；
- 备选供方机制。

### Step 14 — 目标价格

优先依据：

1. 历史成交价；
2. 预算价；
3. 当前市场参考价；
4. 开标后的对标价机制；
5. 采购员确认的降本目标。

模板中的：

- “基本目标降本3%”
- “挑战目标降本5%”

只能作为示例，**不得自动继承**。

### Step 15 — 定标规则

这是人工决策字段。

AI可以：

- 将已确认规则整理成正式报告语言；
- 检查规则是否与需求清单/邀标条件冲突。

AI不能：

- 自行决定中标家数；
- 自行设计淘汰阈值；
- 自行决定最低价一定中标。

### Step 16 — 份额划分

只有中标供方 ≥2 家时才需要。

如只有1家：

- 填“确定1家中标供应商，无份额划分”。

如多家：

- 必须由采购员提供/确认分配逻辑。

### Step 17 — 风险防范

风险应来自真实分析。

至少检查：

- 需求预测风险；
- 价格波动风险；
- 单一供应商风险；
- 区域履约风险；
- 关键SKU供应风险；
- 质量/资质风险；
- 付款账期接受风险；
- 数据不完整风险。

每个风险输出：

```text
风险
→ 证据
→ 影响
→ 建议措施
→ 是否需人工确认
```

### Step 18 — 评标小组成员

只能来自当前项目正式人员安排。

不得复制历史报告中的人员结构或姓名。

## 6. Report Field State

每个主要字段必须内部标记：

- `FACT`
- `CALCULATED`
- `PUBLIC_MARKET_INTELLIGENCE`
- `HUMAN_DECISION`
- `MISSING`

生成报告时不需要展示标签，但必须保留在 `strategy-data.yaml` 中。

## 7. Consistency Gates

输出前必须检查：

### Quantity consistency

报告的预计采购量：

= 已确认需求清单数量。

不得使用过期历史版本。

### Budget consistency

报告预算金额必须与：

- 当前预算；
- 或当前需求量×参考价测算；

保持一致。

### Region consistency

报告配送范围：

= 最终需求清单/邀标条件。

### Shortlist consistency

报告短名单：

= `shortlist-approval` 已确认名单。

### Commercial terms consistency

交期、账期、税率、保证金、合作期限：

必须与当前项目条件一致。

### Award rule consistency

供应商选择方法、定标规则、中标家数、份额划分之间不能互相矛盾。

## 8. Output

### A. Procurement strategy data

`{{项目名称}}-strategy-data.yaml`

包含：

- 全部报告字段；
- 数据来源；
- evidence；
- confidence；
- missing items；
- human decision items。

### B. Procurement plan report

`{{项目名称}}-采购方案报告.docx`

必须：

- 基于 `templates/物资采购方案报告模板.docx`；
- 尽量保留企业原版式；
- 不修改固定制度性页眉/编号等模板元素，除非用户要求；
- 报批前进行人工审核。

## 9. Draft Status

### `draft`

存在：

- MISSING；
- 未确认预算；
- 未确认短名单；
- 未确认定标规则；
- 未确认保证金等关键字段。

### `ready_for_approval`

只有当：

- 关键字段全部有事实/明确人工决策；
- 一致性检查通过；
- 无P0阻断项；

才能标记。

## 10. Guardrails

- 不复用历史案例中的具体项目数据。
- 不把模板示例值当企业当前规则。
- 不虚构市场成本比例。
- 不从公网新增候选供方。
- 不把市场公开供应商等同于邀标候选供方。
- 不覆盖采购员确认的需求量或短名单。
- 不混用含税/未税金额。
- 不把部分周期历史数据当年度数据。
- 不自行确定预算、保证金、降本率、定标规则、份额或评标人员。
- 所有重要结论必须可追溯。

## 11. Success Criteria

1. 报告结构与企业模板一致；
2. 报告主要数据都能追溯到前序 Skill 或明确来源；
3. 历史采购分析真正进入支出分析和需求背景，而不是被闲置；
4. 短名单与采购员确认结果完全一致；
5. 市场信息不会污染官方供方候选池；
6. 报告中的数量、预算、区域和商务条件内部一致；
7. 采购员只需要做关键决策确认，而不是重新整理整份报告。
