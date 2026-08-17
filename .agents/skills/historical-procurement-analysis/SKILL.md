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

至少需要：

- 历史订单数据文件；
- 协议编号字段或可明确映射的同义字段；
- 订单状态；
- 订单日期；
- SKU/物料名称；
- 数量；
- 单位。

协议字段可以包括：

- 协议编号
- 协议号
- 合同编号
- 合同号
- 框架协议编号
- 采购协议编号

不得仅凭文件名、项目名或供应商名称猜协议编号。

## 3. Workflow

### Step 1 — Detect Data Structure

识别协议编号、订单编号、订单状态、订单日期、物料编码/名称、品牌、规格、数量、单位、价格、金额、区域/收货地址、供应商等字段。

### Step 2 — Agreement Selection Gate（强制）

在任何正式历史统计、预测和需求清单数量生成前，必须先提取全部唯一协议编号。

对每个协议编号展示：

| 协议编号 | 原始订单数 | 有效订单数 | 有效明细行 | 数据期间 | SKU数 | 有效含税金额 | 主要区域 |
|---|---:|---:|---:|---|---:|---:|---|

其中“有效”仅用于概览，定义为订单状态属于：

- `已完成`
- `执行中`

然后要求用户**明确选择一个或多个协议编号**。

规则：

1. 未经用户确认不得进入正式分析；
2. 禁止默认选择最新协议；
3. 禁止默认选择金额最大的协议；
4. 禁止默认选择全部协议；
5. 即使只识别到一个协议，也必须让用户确认；
6. 用户可以同时选择多个协议；
7. 多协议分析必须保留 `agreement_number` 维度，先分别统计，再生成合并结果；
8. 无协议编号记录默认排除，只有用户明确要求“包含未关联协议订单”并确认后才能纳入。

如果协议编号字段缺失，标记 `missing_agreement_field`，不进入正式分析。

详细规则见 `references/agreement-selection-rules.md`。

### Step 3 — Apply Selected Agreement Scope

正式数据集必须满足：

`agreement_number IN user_selected_agreement_numbers`

输出范围审计：可选协议数、选中协议数、选中协议编号、未选协议订单数、无协议编号记录数。

### Step 4 — Order Status Filter Gate（强制）

在所选协议范围内，仅保留：

- `已完成`
- `执行中`

排除：

- `已取消`
- `已退货`
- `取消`
- `退货`
- `已作废`
- `关闭`
- 以及其他未明确等于“已完成/执行中”的状态。

取消、退货等记录不得进入采购量、采购金额、平均单价、SKU占比、供应商份额或预测。

输出：原始行数、保留行数、剔除行数、保留订单数、按状态分类的剔除数量。

如果没有订单状态字段，不得默认全部有效；只有采购员明确确认源数据已按上述状态预过滤后才能继续。

### Step 5 — Period Completeness Gate

- 完整年度/可比同期：可进入同比和预测；
- 部分月份/周度：只做 observed baseline、采购结构和价格基线；
- 禁止机械使用 `样本量 ÷ 天数 × 365` 形成年度需求量，除非用户明确要求并接受该假设。

### Step 6 — SKU / Region Normalization

SKU优先使用：

`物料编码 + 规格 + 单位`

无稳定物料编码时使用：

`名称 + 品牌 + 规格/型号 + 单位`

多区域历史订单必须按：

`协议编号 + SKU + 区域`

聚合数量和金额。

如果需求清单同SKU重复行未明确区域，不自动分配数量。

### Step 7 — Historical Analysis

基于所选协议的有效订单分析：

- 历史采购量/金额；
- 历史成交价；
- SKU占比及Top SKU；
- 区域结构；
- 月度/季度趋势；
- 同比数量/金额变化（有可比数据时）；
- 供应商集中度（数据支持时）。

每项结论必须能回溯到协议编号、订单状态和数据期间。

### Step 8 — Projection

有可靠完整基线和明确业务规模增长依据时：

`预计采购量 = 历史基准量 × (1 + 数量增长率)`

如有独立价格变化依据：

`预计单价 = 历史参考单价 × (1 + 价格变化率)`

`预计采购金额 = 预计采购量 × 预计单价`

数量增长与价格增长必须分开。没有规模依据时不得自行假设5%、8%、10%等增长率。

### Step 9 — Requirement List Output（固定）

使用 `templates/需求清单模板.xlsx` 生成：

`{{项目名称}}-历史测算需求清单.xlsx`

可回填采购方字段：区域、名称、品牌、规格、单位、起订量（有明确依据时）、年度预计采购量（预测条件满足时）。

以下供方报价字段必须保持空白：

- 含税单价
- 未税单价
- 税率
- 含税总价

历史价格只进入内部分析和 `strategy_report_handoff`。

预测条件不足时仍可生成需求清单结构，但年度预计采购量留空/待人工确认，不机械年化。

### Step 10 — Existing Requirement Conflict

如果原需求清单已有数量，不静默覆盖。输出：原数量、历史建议量、差异数量、差异比例和 `human_confirmation_status=pending`。

## 4. Structured Handoff

`requirement_handoff` 至少记录：

- selected_agreement_numbers
- order_status_filter
- item_key
- region
- baseline_period
- baseline_quantity
- growth_rate_applied
- suggested_estimated_quantity
- confidence
- evidence
- human_confirmation_status

`strategy_report_handoff` 至少记录：

- selected_agreement_numbers
- 有效订单口径
- 历史采购量/采购额
- SKU/区域结构
- 历史价格
- 同比分析
- 预测依据
- 风险与数据限制

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
  included_statuses:
    - 已完成
    - 执行中
  raw_record_count: 0
  included_record_count: 0
  excluded_record_count: 0
  excluded_by_status: {}
```

## 6. Guardrails

AI MUST NOT：

- 未经用户选择就默认协议；
- 默认选择全部协议；
- 把不同协议静默混合；
- 默认纳入无协议编号订单；
- 把已取消/已退货订单计入历史采购；
- 在状态字段缺失时默认订单有效；
- 将部分周期数据机械年化；
- 无增长依据时制造增长率；
- 不同规格/单位SKU直接合并；
- 将历史价格填入供方报价字段；
- 用低可信度预测静默覆盖人工数量。

## 7. Human Review

采购员最终确认：分析采用的协议编号、是否纳入未关联协议订单、异常订单处理、SKU映射、区域映射、增长依据、最终需求数量和预计金额。

## 8. Success Criteria

1. 用户明确选择了一个或多个协议编号；
2. 只有所选协议进入正式分析；
3. 只有已完成/执行中订单进入历史基线；
4. 每个结果能追溯到协议编号和有效订单；
5. 不完整周期不机械年化；
6. 能按模板输出需求清单；
7. 历史价格与供方报价字段严格隔离；
8. Handoff可被需求分析和采购策略报告直接消费。
