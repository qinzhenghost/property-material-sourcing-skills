---
name: historical-procurement-analysis
description: 当物业物资采购项目存在历史订单数据时，先识别协议编号并要求用户明确选择一个或多个协议，再仅保留“已完成”和“执行中”订单；对不足12个月的有效历史数据强制折算为12个月年度采购基线，并在缺少两年度可比数据时要求用户补充数据或确认预估增量比例，最终按需求清单模板输出非空的年度预计采购量，同时向采购策略报告提供结构化 Handoff。
metadata:
  version: "0.5.5"
  domain: "property-material-procurement"
  stage: "pre-requirement"
---

# Historical Procurement Analysis

## 1. Purpose

本 Skill 位于需求分析之前：

`历史订单 → 协议范围选择 → 有效订单过滤 → 12个月年化基线 → 年度增量确认/计算 → 历史测算需求清单 → Handoff`

主要输出：

1. 协议编号选择摘要；
2. 订单状态过滤审计；
3. SKU / 区域历史采购量、金额和价格分析；
4. 不足12个月数据的12个月年化数量/金额基线；
5. 年度同步增量比例及其依据；
6. 下一年度预计采购量/金额；
7. `requirement_handoff`；
8. `{{项目名称}}-历史测算需求清单.xlsx`；
9. `strategy_report_handoff`。

## 2. Required Inputs

至少需要：历史订单数据文件、协议编号字段或可明确映射的同义字段、订单状态、订单日期、SKU/物料名称、数量、单位。

## 3. Workflow

### Step 1 — Detect Data Structure
识别协议编号、订单编号、订单状态、订单日期、物料编码/名称、品牌、规格、数量、单位、价格、金额、区域/收货地址、供应商等字段。

### Step 2 — Agreement Selection Gate（强制）
提取全部唯一协议编号，展示每个协议的原始订单数、有效订单数、有效明细行、数据期间、SKU数、有效含税金额和主要区域。必须由用户明确选择一个或多个协议编号；禁止默认选择最新协议、金额最大协议或全部协议。多选时保留 `agreement_number` 维度。详见 `references/agreement-selection-rules.md`。

### Step 3 — Apply Selected Agreement Scope
正式数据集满足 `agreement_number IN user_selected_agreement_numbers`。

### Step 4 — Order Status Filter Gate（强制）
仅保留 `已完成`、`执行中`；取消、退货、作废、关闭及其他状态全部排除，不得进入数量、金额、价格、占比、年化或预测。

### Step 5 — Coverage & 12-Month Annualization Gate（强制）
完整12个月：`annualization_factor = 1`，完整年度实际量直接作为年化基线。

不足12个月：必须折算为12个月采购基线，年度预计采购量不得留空。

完整自然月数据：
`annualization_factor = 12 / covered_months`

首尾月不完整/明细日期：
`covered_month_equivalent = covered_days / 365.2425 × 12`
`annualization_factor = 12 / covered_month_equivalent`

然后：
`annualized_baseline_quantity = observed_valid_quantity × annualization_factor`
`annualized_baseline_amount = observed_valid_amount × annualization_factor`

必须记录覆盖周期、年化系数及季节性/集中采购风险。年化值是比例外推，不是历史全年实际。详见 `references/annualization-and-growth-rules.md`。

### Step 6 — SKU / Region Normalization
SKU优先 `物料编码 + 规格 + 单位`，多区域按 `协议编号 + SKU + 区域` 聚合。

### Step 7 — Historical Analysis
分析历史采购量/金额、12个月年化量/金额、成交价、SKU占比、区域结构、趋势、同比及供应商集中度。

### Step 8 — Annual Growth Rate Decision Gate（强制）
下一年度预计采购量基于：
`12个月年化基线 × (1 + 年度增量比例)`

增量比例优先级：
1. 可比两年度数量同比；
2. 可比两年度采购金额同比 `spend_yoy_proxy`；
3. 企业正式业务规模/预算增幅；
4. 用户明确确认的预估增量比例；
5. 用户确认0%增量。

两年度采购金额同比：
`spend_yoy_rate = (later_year_spend - earlier_year_spend) / earlier_year_spend`

金额同比只作为采购规模代理，若价格变化明显必须提示；不得同时作为数量增长和价格增长重复放大。

如果缺少至少两个可比年度数据，必须提醒用户选择：
1. 补充另一年度同口径数据；
2. 提供预估增量比例；
3. 按0%增量。

用户未确认前，需求清单年度预计采购量仍不得留空：先填 `annualized_baseline_quantity`，标记 `provisional_annualized_baseline`，备注“当前为12个月年化基线，年度增量比例待确认”，并设置 `pending_growth_confirmation`。

确认后：
`final_estimated_quantity = annualized_baseline_quantity × (1 + confirmed_growth_rate)`

### Step 9 — Price & Amount Projection
价格增长必须有独立依据。无独立依据时价格增长率默认为0仅用于内部金额测算。不得用同一个金额同比同时放大数量和价格。

### Step 10 — Requirement List Output（固定）
使用 `templates/需求清单模板.xlsx` 生成 `{{项目名称}}-历史测算需求清单.xlsx`。

`年度预计采购量` 必须有值：增长率已确定时填最终预计量；增长率未确定时填12个月年化基线并备注待确认增量比例。

供方报价字段 `含税单价/未税单价/税率/含税总价` 保持空白。

### Step 11 — Existing Requirement Conflict
原需求清单已有数量时不静默覆盖，输出原数量、12个月年化基线、增量比例、新建议量和差异。

## 4. Structured Handoff
`requirement_handoff.quantity_updates` 记录 observed_quantity、covered_month_equivalent、annualization_factor、annualized_baseline_quantity、growth_rate_source、growth_rate_applied、suggested_estimated_quantity、quantity_status、confidence、evidence 和 human_confirmation_status。

`strategy_report_handoff` 记录协议范围、有效订单口径、覆盖周期、年化方法、12个月年化量/金额、增量依据、历史价格和风险。

## 5. Guardrails
AI MUST NOT：未经用户选择默认协议；纳入无效订单；把部分周期直接当全年实际；缺两年度可比数据时自行制造增量比例；把金额同比同时当数量增长和价格增长；将年度预计采购量留空；把临时年化基线冒充最终确认需求；将历史价格写入供方报价字段；静默覆盖人工数量。

## 6. Human Review
采购员最终确认协议编号、订单过滤、SKU/区域映射、年化口径、是否补充另一年度数据、年度增量比例、最终年度预计采购量和预计采购金额。

## 7. Success Criteria
1. 用户明确选择协议；2. 仅有效订单参与；3. 不足12个月一定年化；4. 年度预计采购量不为空；5. 缺两年度数据时明确提示选择；6. 增量未确认时先填年化基线；7. 确认后更新最终预计量；8. Handoff可被下游消费。
