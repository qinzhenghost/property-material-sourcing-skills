---
name: historical-procurement-analysis
description: 当物业物资采购项目存在历史订单数据时，先识别协议编号并要求用户明确选择一个或多个协议，再仅保留“已完成”和“执行中”订单，剔除取消/退货等无效数据，分析历史采购量、金额、价格、区域及采购规模变化，形成可追溯的需求数量/金额基线，并按需求清单模板输出历史测算需求清单，同时向采购策略报告提供结构化 Handoff。
metadata:
  version: "0.5.4"
  domain: "property-material-procurement"
  stage: "pre-requirement"
---

# Historical Procurement Analysis

## 1. Purpose

本 Skill 位于需求分析之前，用于：

`历史订单 → 协议范围选择 → 有效订单过滤 → 历史采购分析 → 需求数量/金额基线 → 历史测算需求清单 → Handoff`

主要输出：

1. 协议编号选择摘要；
2. 订单状态过滤审计；
3. SKU / 区域历史采购量、金额和价格分析；
4. 条件满足时的下一周期建议采购量/金额；
5. `requirement_handoff`；
6. `{{项目名称}}-历史测算需求清单.xlsx`；
7. `strategy_report_handoff`。

## 2. Required Inputs

至少需要历史订单数据、协议编号或可明确映射的同义字段、订单状态、订单日期、物料、数量和单位。

协议字段可以包括：协议编号、协议号、合同编号、合同号、框架协议编号、采购协议编号。不得通过文件名或项目名猜协议编号。

## 3. Workflow

### Step 1 — Detect Data Structure
识别协议编号、订单编号、状态、日期、SKU、数量、单位、价格、金额、区域及供应商。

### Step 2 — Agreement Selection Gate（强制）
先提取全部唯一协议编号，并展示每个协议的原始订单数、有效订单数、有效明细行、数据期间、SKU数、有效含税金额和主要区域。随后要求用户明确选择一个或多个协议编号。

禁止默认选择最新协议、金额最大协议或全部协议；即使只识别到一个协议，也必须让用户确认。多协议分析保留 `agreement_number` 维度，先分别统计再合并。无协议编号记录默认排除，只有用户明确确认后才能纳入。协议字段缺失时不进入正式分析。

详细规则见 `references/agreement-selection-rules.md`。

### Step 3 — Apply Selected Agreement Scope
正式数据集必须满足 `agreement_number IN user_selected_agreement_numbers`，并输出可选协议数、选中协议、未选订单数和无协议编号记录数。

### Step 4 — Order Status Filter Gate（强制）
在所选协议中仅保留 `已完成`、`执行中`；排除已取消、已退货、取消、退货、已作废、关闭及其他状态。被排除记录不得进入数量、金额、价格、占比或预测。输出完整过滤审计。状态字段缺失时不得默认全部有效。

### Step 5 — Period Completeness Gate
完整年度/可比同期可进入同比和预测；部分月份/周度只能做 observed baseline、结构和价格基线，禁止机械年化。

### Step 6 — SKU / Region Normalization
优先以 `物料编码 + 规格 + 单位` 识别SKU；无物料编码时使用 `名称 + 品牌 + 规格/型号 + 单位`。多区域按 `协议编号 + SKU + 区域` 聚合。重复SKU需求行未标区域时不自动分配数量。

### Step 7 — Historical Analysis
分析历史采购量/金额、成交价、SKU占比、Top SKU、区域结构、月度/季度趋势、同比变化及供应商集中度（数据支持时）。结果必须可追溯到协议编号、状态和期间。

### Step 8 — Projection
有完整可靠基线和明确规模增长依据时：`预计采购量 = 历史基准量 × (1 + 数量增长率)`。有独立价格依据时再计算预计单价和预计金额。数量增长与价格增长分开；无依据不自行假设增长率。

### Step 9 — Requirement List Output
使用 `templates/需求清单模板.xlsx` 生成 `{{项目名称}}-历史测算需求清单.xlsx`。可填区域、名称、品牌、规格、单位、起订量（有依据时）和年度预计采购量（预测条件满足时）。含税单价、未税单价、税率、含税总价必须留空。预测条件不足时数量留空/待确认。

### Step 10 — Existing Requirement Conflict
已有人工数量时不覆盖，输出原数量、建议量、差异数量、差异比例和待人工确认状态。

## 4. Structured Handoff

`requirement_handoff` 至少记录 selected_agreement_numbers、order_status_filter、item_key、region、baseline_period、baseline_quantity、growth_rate_applied、suggested_estimated_quantity、confidence、evidence 和 human_confirmation_status。

`strategy_report_handoff` 至少记录选中协议、有效订单口径、历史采购量/额、SKU/区域结构、价格、同比、预测依据和数据限制。

## 5. Mandatory Audits

```yaml
agreement_selection:
  agreement_field: 协议编号
  available_agreements: []
  selected_agreement_numbers: []
  selection_confirmed: true
  include_unlinked_agreement_records: false
  unselected_order_count: 0
  unlinked_agreement_record_count: 0
order_status_filter:
  included_statuses: [已完成, 执行中]
  raw_record_count: 0
  included_record_count: 0
  excluded_record_count: 0
  excluded_by_status: {}
```

## 6. Guardrails

不得未经用户选择默认协议，不得默认选择全部协议，不得静默混合不同协议，不得默认纳入无协议编号订单，不得统计取消/退货订单，不得机械年化部分周期，不得无依据制造增长率，不得把历史价格填入供方报价列，不得静默覆盖人工数量。

## 7. Human Review

采购员确认：分析协议编号、是否纳入未关联协议订单、异常订单、SKU/区域映射、增长依据、最终需求数量和预计金额。

## 8. Success Criteria

用户明确选择协议范围；只有所选协议且已完成/执行中订单进入历史基线；结果可追溯；部分周期不机械年化；需求清单按模板生成；历史价格与供方报价严格隔离。
