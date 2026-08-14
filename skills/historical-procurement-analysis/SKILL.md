---
name: historical-procurement-analysis
description: 当物业物资采购项目存在历史采购数据时，分析历史采购量、金额、单价及采购规模变化，形成可追溯的下一周期需求数量/金额基线，并把结果交给需求清单和采购策略报告。不得在没有依据时自行制造增长率或采购量。
metadata:
  version: "0.5.0"
  domain: "property-material-procurement"
  stage: "pre-requirement"
---

# Historical Procurement Analysis

## 1. Purpose

当用户提供历史采购数据时，在正式生成/澄清需求清单之前先完成历史采购分析。

本 Skill 的目标不是做复杂预测模型，而是：

> 历史实际数据 + 明确的采购规模变化 + 可解释公式 → 需求数量/金额基线。

主要产出：

1. 历史采购数据摘要；
2. SKU 级历史采购量/金额基线；
3. 同比采购规模变化；
4. 下一周期建议采购量；
5. 下一周期建议采购金额；
6. 可回填需求清单的 `requirement_handoff`；
7. 可直接供采购策略报告使用的 `strategy_report_handoff`。

## 2. When to Use

当满足任一条件时使用：

- 用户提供上一年度或多年度采购明细；
- 用户提供历史采购量/金额汇总；
- 用户说明该项目属于续采、框架续签或已有成熟采购记录；
- 用户希望根据历史采购规模估算下一年度需求。

如果完全没有历史采购数据：

- 不强制执行本 Skill；
- 直接进入 `material-requirement-analysis`；
- 不得自行编造历史采购量或增长率。

## 3. Inputs

至少需要：

- 历史采购数据文件；
- SKU/物料名称；
- 单位；
- 至少一个可作为基线的完整采购周期。

可选：

- 品牌；
- 规格；
- 区域；
- 项目；
- 供应商；
- 含税/未税金额；
- 历史单价；
- 历史采购规模；
- 下一周期采购规模/业务规模；
- 企业年度预算或业务计划。

## 4. Core Principle

### Facts before forecast

所有预测必须明确区分：

- `historical_actual`：历史实际；
- `calculated`：基于公式计算；
- `assumption`：采购员/业务计划提供的假设；
- `human_confirmed`：采购员最终确认。

不得把预测写成历史事实。

## 5. Workflow

### Step 1 — Detect Historical Data

识别：

- 数据年度/月份；
- SKU；
- 数量；
- 金额；
- 单价；
- 单位；
- 区域/项目；
- 供应商。

确认是否存在至少一个完整周期。

### Step 2 — Normalize SKU Identity

跨年度比较必须尽量确认是同一 SKU。

稳定键优先使用：

`名称 + 品牌 + 规格/型号 + 单位`

如果只有名称但规格变化：

- 不直接合并；
- 标记 `sku_comparability_gap`；
- 要求采购员确认。

### Step 3 — Normalize Units and Price Basis

不得直接合并：

- 箱 vs 个；
- 桶 vs 升；
- 含税金额 vs 未税金额；
- 20只/箱 vs 30只/箱。

如果无法标准化：

- 保留原始值；
- 标记不可直接同比；
- 不强制生成该 SKU 的数量预测。

### Step 4 — Build Historical Baseline

优先级：

1. 最近一个完整年度实际量；
2. 若最近年度明显异常，可使用多年度平均，但必须说明；
3. 若只有金额没有可靠数量，则只能做 `amount_only` 预测；
4. 一次性异常采购不得默认作为常态基线。

输出每个 SKU：

- baseline_period
- baseline_quantity
- baseline_amount
- baseline_unit_price
- evidence

### Step 5 — Calculate Historical Trends

有两年以上数据时计算：

- 数量同比；
- 金额同比；
- 平均单价变化；
- SKU 占比；
- Top SKU；
- 月度/季度波动（如数据支持）；
- 供应商采购集中度（如数据支持）。

只分析存在的数据，不为了报告完整而补造指标。

### Step 6 — Analyze Procurement Scale

如果用户提供：

- 项目数量；
- 管理面积；
- 服务户数；
- 采购点位数；
- 业务规模；
- 企业正式年度计划；

可计算：

`规模增长率 = (本期规模 - 上期规模) / 上期规模`

必须保存证据。

如果没有明确采购规模依据：

- `growth_basis = unavailable`
- 不得默认增长 5%、8%、10% 等。

### Step 7 — Select Projection Method

#### A. Quantity-driven（优先）

当 SKU 历史数量可靠时：

`建议采购量 = 历史基准量 × (1 + 规模增长率)`

如果还有采购员明确的 SKU 特殊调整：

`最终建议量 = 规模调整后数量 × (1 + 人工调整率)`

必须分开记录两个调整因子。

#### B. Amount-only

只有历史金额可靠、数量不可比时：

`建议采购金额 = 历史基准金额 × (1 + 规模增长率)`

此时：

- 可以输出项目金额预测；
- 不得为了填需求清单而反推虚假的 SKU 数量。

#### C. No projection

如果历史数据不完整且规模依据缺失：

- 只输出历史分析；
- 不生成预测数量。

### Step 8 — Separate Quantity Growth from Price Growth

严禁重复计算增长。

数量增长：

`projected_quantity = baseline_quantity × (1 + quantity_growth_rate)`

价格增长：

`projected_unit_price = baseline_unit_price × (1 + price_growth_rate)`

最终金额：

`projected_amount = projected_quantity × projected_unit_price`

如果没有独立价格上涨依据：

`price_growth_rate = 0`

不得把同一个 8% 同时重复加到数量和金额。

### Step 9 — Generate Requirement Handoff

对有可靠预测的 SKU，生成：

```yaml
quantity_updates:
  - item_key: ...
    quantity_source: historical_projected
    baseline_quantity: ...
    baseline_period: ...
    growth_rate_applied: ...
    suggested_estimated_quantity: ...
    unit: ...
    suggested_estimated_amount: ...
    confidence: ...
    evidence:
      - ...
    human_confirmation_status: pending
```

下游 `material-requirement-analysis` 应把：

`suggested_estimated_quantity`

写入/映射到需求清单的：

`estimated_quantity / 年度预计采购量`

同时保留来源和人工确认状态。

### Step 10 — Fill Requirement Quantity Row

当用户同时提供待生成/待修改的需求清单时：

- 匹配同一 SKU；
- 将可靠的建议数量填入“预计采购量/年度预计采购量”；
- 若表内已有人工填写数量，不直接覆盖；
- 输出冲突：原数量、历史建议数量、差异比例，由采购员确认。

金额字段同理：

- 有明确数量和单价依据时可生成；
- 无可靠单价时不自行估价。

### Step 11 — Generate Strategy Report Handoff

生成可用于未来 `sourcing-strategy` 的：

- 年度采购金额；
- 年度采购数量；
- 同比数量变化；
- 同比金额变化；
- 单价趋势；
- Top SKU / SKU 集中度；
- 供应商集中度（如有）；
- 采购规模增长依据；
- 下一周期数量/金额测算依据；
- 异常采购与风险；
- 可直接写入报告的事实句。

### Step 12 — Confidence

#### High

- 2年以上完整数据；
- SKU匹配清晰；
- 单位/税口径一致；
- 规模增长有正式依据。

#### Medium

- 仅1个完整年度；
- 或部分SKU需要人工映射；
- 但数量和规模依据基本明确。

#### Low

- 数据缺失较多；
- SKU规格变化明显；
- 单位不可完全统一；
- 规模增长依据弱。

低可信度结果可以作为参考，但不得自动覆盖需求数量。

## 6. Output Format

### A. 历史采购摘要

- 分析周期
- 历史采购量
- 历史采购金额
- 同比变化
- 主要 SKU
- 数据质量

### B. 数量/金额预测表

| SKU | 历史基准量 | 规模增幅 | 建议采购量 | 参考单价 | 建议金额 | 可信度 |
|---|---:|---:|---:|---:|---:|---|

### C. 需求清单回填建议

明确哪些行：

- 可自动填入；
- 需要人工确认；
- 不具备可靠预测条件。

### D. 采购策略报告 Handoff

只保留事实和可追溯计算。

## 7. Guardrails

AI MUST NOT：

- 没有历史数据却制造历史采购量；
- 没有业务规模依据却自动假设增长率；
- 把采购金额增长率直接等同于采购数量增长率；
- 不同 SKU / 不同规格直接合并；
- 不同单位直接累计；
- 含税/未税金额混合后直接同比；
- 将一次性异常采购作为正常基线而不提示；
- 同一个增长率同时重复加到数量和金额；
- 在无单价依据时自行编造预计单价；
- 用低可信度预测静默覆盖采购员已填写的数量。

## 8. Human Review

必须由采购员最终确认：

- SKU 映射；
- 异常数据是否剔除；
- 采购规模增长率；
- 特殊 SKU 调整；
- 最终预计采购量；
- 最终预计采购金额。

## 9. Structured Handoff

### Downstream 1

`material-requirement-analysis`

用于：

- 回填预计采购数量；
- 提供数量来源；
- 识别与采购员原始数量的冲突；
- 继续完成规格和商务条件澄清。

### Downstream 2

`sourcing-strategy`

用于：

- 支出分析；
- 历史采购趋势；
- 需求背景；
- 预算/采购金额测算依据；
- 风险分析。

## 10. Success Criteria

1. 每个预测数量能回溯到历史实际数据；
2. 增长率来源明确；
3. 不重复计算增量；
4. 能给需求清单提供数量基线；
5. 能给采购策略报告提供事实化分析；
6. 采购员能清楚区分历史事实、AI计算与人工确认值。
