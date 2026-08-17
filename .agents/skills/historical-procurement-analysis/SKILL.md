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

至少需要：

- 历史订单数据文件；
- 协议编号字段或可明确映射的同义字段；
- 订单状态；
- 订单日期；
- SKU/物料名称；
- 数量；
- 单位。

协议字段可包括：协议编号、协议号、合同编号、合同号、框架协议编号、采购协议编号。

不得仅凭文件名、项目名或供应商名称猜协议编号。

## 3. Workflow

### Step 1 — Detect Data Structure

识别协议编号、订单编号、订单状态、订单日期、物料编码/名称、品牌、规格、数量、单位、价格、金额、区域/收货地址、供应商等字段。

### Step 2 — Agreement Selection Gate（强制）

在任何正式历史统计、预测和需求清单数量生成前，提取全部唯一协议编号。

对每个协议编号展示：

| 协议编号 | 原始订单数 | 有效订单数 | 有效明细行 | 数据期间 | SKU数 | 有效含税金额 | 主要区域 |
|---|---:|---:|---:|---|---:|---:|---|

“有效”仅指订单状态为 `已完成` 或 `执行中`。

必须由用户明确选择一个或多个协议编号。禁止默认选择最新协议、金额最大协议或全部协议；即使只有一个协议也必须确认。

多选时必须保留 `agreement_number` 维度，先分别统计，再生成合并结果。无协议编号记录默认排除，除非用户明确确认纳入。

详见 `references/agreement-selection-rules.md`。

### Step 3 — Apply Selected Agreement Scope

正式数据集必须满足：

`agreement_number IN user_selected_agreement_numbers`

输出协议范围审计。

### Step 4 — Order Status Filter Gate（强制）

仅保留：

- `已完成`
- `执行中`

排除取消、退货、作废、关闭及其他状态。

这些被排除记录不得进入数量、金额、平均单价、SKU占比、供应商份额、年化或预测。

如果无订单状态字段，不得默认全部有效；只有采购员明确确认源数据已预过滤后才能继续。

### Step 5 — Coverage & 12-Month Annualization Gate（强制）

订单状态过滤完成后，判断所选协议有效数据覆盖周期。

#### 5.1 完整12个月或完整年度

若存在完整12个月可比历史：

- 直接把该完整年度实际量作为 `annualized_baseline_quantity`；
- `annualization_factor = 1`；
- 不重复放大。

#### 5.2 不足12个月

如果有效历史数据不足12个月，**必须折算为12个月采购基线**，不得把年度预计采购量留空。

优先计算方式：

1. 如果数据按完整自然月提供，使用实际覆盖月数 `covered_months`；
2. 如果首尾月份不完整或是订单明细日期，使用有效数据起止日期计算 `covered_days`，再换算月当量：
   `covered_month_equivalent = covered_days / 365.2425 × 12`；
3. `annualization_factor = 12 / covered_month_equivalent`；
4. `annualized_baseline_quantity = observed_valid_quantity × annualization_factor`；
5. `annualized_baseline_amount = observed_valid_amount × annualization_factor`。

必须保留：

- 数据起止日期；
- 覆盖天数/月当量；
- 年化系数；
- 原始有效采购量/金额；
- 年化后的12个月采购量/金额；
- `annualization_method`；
- 数据季节性/集中下单风险提醒。

年化结果是**基于比例外推的年度基线**，不是历史实际全年值。

如果数据少于1个月或明显属于一次性集中采购，仍按用户要求提供年化基线，但 `confidence=low` 并突出风险。

详见 `references/annualization-and-growth-rules.md`。

### Step 6 — SKU / Region Normalization

SKU优先使用：

`物料编码 + 规格 + 单位`

无稳定物料编码时使用：

`名称 + 品牌 + 规格/型号 + 单位`

多区域订单按：

`协议编号 + SKU + 区域`

聚合数量和金额。

### Step 7 — Historical Analysis

基于所选协议有效订单分析：

- 历史采购量/金额；
- 年化12个月采购量/金额；
- 历史成交价；
- SKU占比及Top SKU；
- 区域结构；
- 月度/季度趋势；
- 同比数量/金额变化；
- 供应商集中度。

### Step 8 — Annual Growth Rate Decision Gate（强制）

下一年度预计采购量必须基于：

`12个月年化基线 × (1 + 年度增量比例)`

年度增量比例优先级：

1. **可比两年度数量同比**：SKU数量口径可靠时优先；
2. **可比两年度采购金额同比**：当数量口径不可比或用于整体采购规模时，可计算采购金额同比作为 `spend_yoy_proxy`；
3. 企业正式业务规模/预算增幅；
4. 用户明确确认的预估增量比例；
5. 用户明确选择 `0%` 增量，直接使用12个月年化基线。

两年度采购金额同比：

`spend_yoy_rate = (later_year_spend - earlier_year_spend) / earlier_year_spend`

注意：

- 金额同比只作为采购规模代理，不等于价格增长；
- 如果能识别明显价格变化，必须提示金额同比可能包含价格因素；
- 数量增长和价格增长仍需分开，避免双重放大。

#### 缺少两年度可比数据时

如果无法从至少两个可比年度的有效采购金额/数量测算年度同步增量比例，Skill 必须提醒用户并给出三个明确选项：

1. **补充数据**：补充另一年度同口径历史订单/年度采购金额；
2. **提供预估增量比例**：例如 `+5%`、`+8%`、`-3%`；
3. **按0%增量**：直接采用12个月年化基线作为年度预计采购量。

在用户确认前：

- 需求清单中的 `年度预计采购量` **不得留空**；
- 先填入 `annualized_baseline_quantity` 作为 `provisional_annualized_baseline`；
- 备注必须写明“当前为12个月年化基线，年度增量比例待确认”；
- `human_confirmation_status = pending_growth_confirmation`；
- 不得把该临时基线描述为最终确认需求量。

用户选择后重新计算并更新为：

`final_estimated_quantity = annualized_baseline_quantity × (1 + confirmed_growth_rate)`

### Step 9 — Price & Amount Projection

有独立价格变化依据时：

`projected_unit_price = baseline_unit_price × (1 + price_growth_rate)`

`projected_amount = final_estimated_quantity × projected_unit_price`

无独立价格变化依据时，价格增长率默认为0仅用于内部金额测算，并清晰标注。

不得把同一个采购金额同比同时当作数量增长和价格增长重复计算。

### Step 10 — Requirement List Output（固定）

使用 `templates/需求清单模板.xlsx` 生成：

`{{项目名称}}-历史测算需求清单.xlsx`

`年度预计采购量` 必须有值：

- 完整年度且增长率已确定：填最终预计量；
- 不足12个月且增长率已确定：填年化基线 × 增长率后的预计量；
- 增长率尚未确定：填12个月年化基线，并在备注标记“待确认年度增量比例”。

可回填采购方字段：

- 序号
- 区域
- 名称
- 品牌
- 规格
- 单位
- 起订量（有明确依据时）
- 年度预计采购量
- 备注

以下供方报价字段保持空白：

- 含税单价
- 未税单价
- 税率
- 含税总价

历史参考价格只进入内部分析与 `strategy_report_handoff`。

### Step 11 — Existing Requirement Conflict

如果原需求清单已有数量，不静默覆盖。输出：

- 原数量；
- 12个月年化基线；
- 增量比例；
- 新建议量；
- 差异数量/比例；
- 人工确认状态。

## 4. Structured Handoff

`requirement_handoff.quantity_updates` 至少记录：

- item_key
- region
- observed_quantity
- covered_month_equivalent
- annualization_factor
- annualized_baseline_quantity
- growth_rate_source
- growth_rate_applied
- suggested_estimated_quantity
- quantity_status
- confidence
- evidence
- human_confirmation_status

`strategy_report_handoff` 至少记录：

- selected_agreement_numbers
- 有效订单口径
- 覆盖周期
- 年化方法与年化系数
- 原始采购量/采购额
- 12个月年化采购量/金额
- 同比/增量比例依据
- 历史价格
- 风险与数据限制

## 5. Mandatory Audits

```yaml
annualization:
  coverage_start: 2025-01-01
  coverage_end: 2025-06-30
  covered_days: 181
  covered_month_equivalent: 5.95
  annualization_method: day_equivalent_to_12_months
  annualization_factor: 2.0168
  annualization_required: true

growth_rate_decision:
  status: pending_user_confirmation
  two_year_comparable_data_available: false
  calculated_growth_rate: null
  growth_rate_source: unavailable
  user_options:
    - supplement_comparable_history
    - provide_estimated_growth_rate
    - use_zero_growth
```

## 6. Guardrails

AI MUST NOT：

- 未经用户选择就默认协议；
- 纳入取消/退货等无效订单；
- 把部分周期采购量直接当全年历史实际；
- 年化后不说明比例外推假设；
- 缺两年度可比数据时自行制造增量比例；
- 把采购金额同比同时当数量增长和价格增长；
- 将年度预计采购量留空；
- 将临时12个月年化基线冒充最终人工确认需求；
- 不同规格/单位SKU直接合并；
- 将历史价格填入供方报价字段；
- 静默覆盖采购员已有数量。

## 7. Human Review

采购员最终确认：

- 协议编号；
- 无效订单过滤；
- SKU/区域映射；
- 年化周期口径；
- 是否补充另一年度数据；
- 年度增量比例；
- 最终年度预计采购量；
- 预计采购金额。

## 8. Success Criteria

1. 用户明确选择一个或多个协议编号；
2. 只有已完成/执行中订单参与计算；
3. 不足12个月的数据一定生成12个月年化采购量；
4. 需求清单年度预计采购量不为空；
5. 缺少两年度可比数据时明确要求用户选择增量处理方式；
6. 增量未确认时，需求清单以12个月年化基线临时填充并清楚标注；
7. 增量确认后可更新为最终预计采购量；
8. Handoff可被需求分析和采购策略报告直接消费。
