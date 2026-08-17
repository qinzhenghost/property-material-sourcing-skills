---
name: historical-procurement-analysis
description: 当物业物资采购项目存在历史采购数据时，先剔除非有效订单，仅保留“已完成”和“执行中”订单，再分析历史采购量、金额、单价及采购规模变化，形成可追溯的下一周期需求数量/金额基线，并输出一份按标准需求清单模板填入的需求清单草稿，同时向采购策略报告提供结构化 Handoff。
metadata:
  version: "0.5.3"
  domain: "property-material-procurement"
  stage: "pre-requirement"
---

# Historical Procurement Analysis

## 1. Purpose

当用户提供历史采购数据时，在正式生成/澄清需求清单之前先完成历史采购分析。

核心目标：

> 有效历史订单 + 明确采购规模变化 + 可解释公式 → 可追溯的需求数量/金额基线。

本 Skill 必须先做订单状态过滤，再做任何统计和预测。

主要产出：

1. 历史订单有效性过滤摘要；
2. 历史采购数据摘要；
3. SKU / 区域级历史采购量与金额基线；
4. 同比采购趋势与采购规模变化；
5. 下一周期建议采购量与建议采购金额；
6. `requirement_handoff`；
7. 一份按标准需求清单模板生成的 `{{项目名称}}-历史测算需求清单.xlsx`；
8. `strategy_report_handoff`。

## 2. When to Use

当满足任一条件时使用：

- 用户提供上一年度或多年度采购明细；
- 用户提供订单导出、历史采购量/金额汇总；
- 项目属于续采、框架续签或已有成熟采购记录；
- 用户希望根据历史采购规模估算下一周期需求。

如果完全没有历史采购数据：

- 跳过本 Skill；
- 直接进入 `material-requirement-analysis`；
- 不得自行编造历史采购量或增长率。

## 3. Required Inputs

至少需要：

- 历史采购数据文件；
- SKU/物料名称；
- 单位；
- 订单状态字段，或采购员明确确认数据已经按有效状态预过滤。

推荐字段：

- 订单编号；
- 订单创建日期；
- 订单状态；
- 物料编码；
- 物料名称；
- 品牌；
- 规格；
- 数量；
- 单位；
- 区域/收货地址；
- 含税/未税单价；
- 含税/未税金额；
- 税率；
- 供应商。

## 4. Core Principle

所有结果必须区分：

- `historical_actual`：过滤后的历史实际；
- `calculated`：基于公式计算；
- `assumption`：采购员/业务计划提供的假设；
- `human_confirmed`：采购员最终确认。

不得把预测写成历史事实。

## 5. Workflow

### Step 1 — Detect Data Structure

识别：

- 订单编号；
- 订单状态；
- 日期；
- SKU；
- 数量；
- 金额；
- 单价；
- 单位；
- 区域/项目；
- 供应商。

### Step 2 — Order Status Filter Gate（强制）

在任何数量、金额、价格、同比或预测计算前，必须先过滤订单状态。

**唯一允许进入历史基线的状态：**

- `已完成`
- `执行中`

以下以及任何其他状态全部排除：

- `已取消`
- `已退货`
- `取消`
- `退货`
- `已作废`
- `关闭`
- 其他不等于“已完成/执行中”的状态。

规则：

1. 状态文本先去除首尾空格、统一全半角等不改变语义的格式；
2. 仅按明确状态值过滤，不根据付款、发货、收货字段猜订单状态；
3. 已取消、已退货记录不得进入任何采购量、采购金额、平均单价、SKU占比或供应商份额计算；
4. 如果同一订单有多条物料行，以订单状态关联其全部明细行；
5. 输出过滤审计：原始行数、保留行数、剔除行数、各剔除状态数量；
6. 如果数据没有订单状态字段：
   - 标记 `missing_status_field`；
   - 不得静默把全部订单当有效；
   - 除非采购员明确确认该数据已预过滤，否则不得形成正式历史基线。

详细规则见：`references/order-status-filter-and-requirement-output.md`。

### Step 3 — Period Completeness Gate

过滤有效订单后再判断周期完整性。

- 完整年度/可比周期：可进入同比和预测；
- 部分月份/周度：只做 observed baseline、结构和价格参考；
- 禁止按 `样本量 ÷ 天数 × 365` 机械年化，除非用户明确要求并接受该假设。

### Step 4 — Normalize SKU Identity

优先使用：

`物料编码 + 规格 + 单位`

无稳定物料编码时使用：

`名称 + 品牌 + 规格/型号 + 单位`

若规格变化或匹配不唯一：

- 不直接合并；
- 标记 `sku_comparability_gap`；
- 请求人工确认。

### Step 5 — Normalize Units and Price Basis

不得直接合并：

- 箱 vs 个；
- 桶 vs 升［
- 20只/箱 vs 30只/箱；
- 含税金额 vs 未税金额。

无法可靠标准化时保留原值并降低可信度。

### Step 6 — Region Mapping

历史订单覆盖多个区域时，必须按：

`SKU + 区域`

聚合数量与金额。

区域优先来源：

1. 地址省份；
2. 地址城市；
3. 收货地址；
4. 使用部门/项目（仅在能明确映射区域时）。

如果需求清单存在同 SKU 重复行但没有区域字段：

- 禁止自动把数量分配给重复行；
- 先建立区域映射。

标准需求清单模板已新增“区域”列。

### Step 7 — Build Historical Baseline

优先级：

1. 最近一个完整年度过滤后的实际量；
2. 若最近年度明显异常，可使用多年度平均，但必须说明；
3. 若只有金额可靠，则使用 `amount_only`［
4. 一次性异常采购不得默认作为常态基线。

输出每个 SKU / 区域：

- baseline_period
- baseline_quantity
- baseline_amount
- baseline_unit_price
- source_order_status
- evidence

### Step 8 — Historical Trends

有两年以上可比数据时计算：

- 数量同比；
- 金额同比［
- 平均单价变化［
- SKU占比［
- Top SKU；
- 月度/季度波动；
- 区域结构；
- 供应商集中度（如数据支持）。

只分析真实存在的数据。

### Step 9 — Analyze Procurement Scale

如果用户提供项目数、管理面积、服务户数、采购点位数或正式业务计划，可计算：

`规模增长率 = (目标期规模 - 基准期规模) / 基准期规模`

必须保存证据。

没有明确依据时：

- `growth_basis = unavailable`
- 不得默认增长 5%、8%、10% 等。

### Step 10 — Select Projection Method

#### A. Quantity-driven

`建议采购量 = 历史基准量 × (1 + 规模增长率)`

如有采购员确认的SKU特殊调整：

`最终建议量 = 规模调整后数量 × (1 + 人工调整率)`

#### B. Amount-only

`建议采购金额 = 历史基准金额 × (1 + 规模增长率)`

只有金额可靠时，不得反推虚假SKU数量。

#### C. No projection

历史周期不完整、SKU不可比或增长依据缺失时：

- 只输出历史分析；
- 年度预计采购量保持待确认；
- 不强行填入预测值。

### Step 11 — Separate Quantity Growth from Price Growth

数量增长：

`projected_quantity = baseline_quantity × (1 + quantity_growth_rate)`

价格增长：

`projected_unit_price = baseline_unit_price × (1 + price_growth_rate)`

最终金额：

`projected_amount = projected_quantity × projected_unit_price`

没有独立价格上涨依据时：

`price_growth_rate = 0`

不得对金额再次重复乘同一个数量增长率。

### Step 12 — Generate Requirement Handoff

对可靠预测的 SKU / 区域生成：

```yaml
quantity_updates:
  - item_key: ...
    region: ...
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

### Step 13 — Generate Filled Requirement Workbook（新增固定输出）

必须基于：

`templates/需求清单模板.xlsx`

生成：

`{{项目名称}}-历史测算需求清单.xlsx`

回填字段：

- 序号；
- 区域［
- 名称［
- 品牌（历史数据明确时）［
- 规格［
- 单位［
- 起订量（只有当前需求或正式依据明确时）［
- 年度预计采购量（只有预测条件满足时）。

**供方报价字段必须保持空白：**

- 含税单价；
- 未税单价；
- 税率；
- 含税总价。

历史参考价格与预计采购金额只进入分析结果和 Handoff，不得写入供方报价列。

如果无法形成可靠年度数量：

- 仍生成需求清单草稿；
- 填入可确认的SKU/区域/规格等字段；
- 年度预计采购量保持空白；
- 备注写明“历史数据不足/待人工确认”。

如果用户同时提供原需求清单且已有人工数量：

- 不静默覆盖；
- 输出原数量、历史建议数量、差异比例；
- 由采购员确认最终值。

### Step 14 — Generate Strategy Report Handoff

输出：

- 有效订单过�[�情况；
- 年度采购金额/数量；
- 同比变化；
- 单价趋势；
- 区域结构；
- Top SKU / SKU集中度［
- 采购规模增长依据［
- 下一周期数量/金额测算依据；
- 异常采购与风险；
- 可直接进入采购策略报告的事实句。

### Step 15 — Confidence

#### High

- 2年以上完整有效订单数据［
- 状态字段完整［
- SKU匹配清晰［
- 单位/税口径一致；
- 规模增长有正式依据。

#### Medium

- 仅1个完整年度；
- 或部分SKU需要人工映射；
- 但状态、数量和规模依据基本明确。

#### Low

- 状态字段不完整［
- 数据缺失较多［
- SKU规格变化明显［
- 单位不可完全统一［
- 规模增长依据弱。

低可信度结果不得静默覆盖采购员已填写数量。

## 6. Output Format

### A. 订单状态过滤摘要

至少包含：

- 原始订单/明细数量［
- 保留订单/明细数量；
- 剔除订单/明细数量；
- 各剔除状态数量；
- 状态字段来源。

### B. 历史采购摘要

- 分析周期［
- 有效历史采购量［
- 有效历史采购金额［
- 同比变化［
- 主要SKU/区域；
- 数据质量。

### C. 数量/金额预测表

| SKU | 区域 | 历史基准量 | 规模增幅 | 建议采购量 | 参考单价 | 建议金额 | 可信度 |
|---|---|---:|---:|---:|---:|---:|---|

### D. Structured Handoff

- `requirement_handoff`
- `strategy_report_handoff`

### E. 需求清单 Excel（新增）

`{{项目名称}}-历史测算需求清单.xlsx`

用于进入 `material-requirement-analysis` 继续做规格、数量和商务条件澄清。

## 7. Guardrails

AI MUST NOT：

- 把已取消、已退货或任何非“已完成/执行中”订单计入历史基线；
- 没有订单状态字段却默认全部有效；
- 没有历史数据却制造历史采购量；
- 没有业务规模依据却自动假设增长率；
- 把采购金额增长率直接等同于采购数量增长率；
- 不同SKU/规格/区域直接合并；
- 不同单位直接累计；
- 含税/未税金额混合同比；
- 将一次性异常采购作为正常基线而不提示；
- 同一个增长率同时重复加到数量和金额；
- 在无单价依据时自行编造预计单价；
- 用历史参考单价预填供方报价列；
- 用低可信度预测静默覆盖采购员已填写数量。

## 8. Human Review

必须由采购员最终确认：

- 订单状态字段及过滤结果；
- SKU/区域映射；
- 异常数据是否剔除；
- 采购规模增长率；
- 特殊SKU调整；
- 最终预计采购量；
- 最终预计采购金额。

## 9. Structured Handoff

### Downstream 1 — material-requirement-analysis

提供：

- 过滤后的历史基线；
- 预计采购数量；
- 数量来源与可信度；
- 新生成的历史测算需求清单；
- 与人工原数量的冲突。

### Downstream 2 — sourcing-strategy

提供：

- 有效订单过滤说明［
- 支出分析［
- 历史趋势［
- 区域/SKU结构；
- 需求背景；
- 预算测算依据；
- 风险分析。

## 10. Success Criteria

1. 所有统计只来自“已完成/执行中”订单；
2. 过滤过程可审计［
3. 每个预测数量能回溯到过滤后的历史实际数据；
4. 增长率来源明确且不重复计算；
5. 能生成一份按标准模板填入的需求清单草稿［
6. 供方报价字段保持空白；
7. 能给采购策略报告提供事实化分析；
8. 采购员能区分历史事实、AI计算与人工确认值。