---
name: historical-procurement-analysis
description: 当物业物资采购项目存在历史订单数据时，先识别协议编号并要求用户明确选择一个或多个协议，再仅保留“已完成”和“执行中”订单；对不足12个月的有效历史数据折算为12个月年度采购基线，并在缺少两年度可比数据时要求用户补充数据或确认预估增量比例；同时输出可直接支撑采购策略报告的多维支出分析 Handoff。
metadata:
  version: "0.5.6"
  domain: "property-material-procurement"
  stage: "pre-requirement"
---

# Historical Procurement Analysis

## 1. Purpose

主流程：

`历史订单 → 协议范围选择 → 有效订单过滤 → 12个月年化基线 → 年度增量确认/计算 → 历史测算需求清单 → Handoff`

主要输出：

1. 协议编号选择摘要；
2. 订单状态过滤审计；
3. SKU / 区域历史采购量、金额和价格分析；
4. 业务单元/军种/业态/项目结构（源数据存在时）；
5. 订单金额段/订单碎片化分析（订单级数据存在时）；
6. 历史供方支出集中度（源数据存在时）；
7. 不足12个月数据的12个月年化数量/金额基线；
8. 年度增量比例及依据；
9. `requirement_handoff`；
10. `{{项目名称}}-历史测算需求清单.xlsx`；
11. `strategy_report_handoff`。

## 2. Required Inputs

至少需要：历史订单数据、协议编号、订单状态、订单日期、SKU/物料名称、数量、单位。

如存在以下字段也必须识别并用于策略报告 Handoff：订单编号、订单金额、区域/收货地址、军种/业务单元/业态/项目/客户线/组织、供应商、品牌/规格、含税/未税单价及税率。

不得仅凭文件名、项目名或供应商名称猜协议编号。

## 3. Workflow

### Step 1 — Detect Data Structure

识别协议编号、订单编号、订单状态、订单日期、物料编码/名称、品牌、规格、数量、单位、价格、金额、区域/收货地址、业务单元/军种/业态/项目、供应商等字段，并记录 `available_analysis_dimensions`。

### Step 2 — Agreement Selection Gate（强制）

任何正式统计、预测和需求清单生成前提取全部唯一协议编号，并展示每个协议的原始订单数、有效订单数、有效明细行、期间、SKU数、有效金额、主要区域。

“有效”只指 `已完成` 或 `执行中`。必须由用户明确选择一个或多个协议编号；禁止默认选择最新、金额最大或全部协议，即使只有一个协议也要确认。多协议保留 `agreement_number` 维度，无协议编号记录默认排除。

### Step 3 — Apply Selected Agreement Scope

正式数据集满足 `agreement_number IN user_selected_agreement_numbers`，输出范围审计。

### Step 4 — Order Status Filter Gate（强制）

仅保留 `已完成 / 执行中`。取消、退货、作废、关闭及其他状态不得进入数量、金额、平均价、占比、年化、预测和策略报告统计。没有订单状态字段时不得静默使用全部数据，除非采购员明确确认已预过滤。

### Step 5 — Coverage & 12-Month Annualization Gate（强制）

完整12个月/完整年度：`annualization_factor = 1`，不重复放大。

不足12个月：必须折算12个月基线。完整自然月优先 `12 / covered_months`；首尾月不完整则 `covered_month_equivalent = covered_days / 365.2425 × 12`，再计算 `annualization_factor = 12 / covered_month_equivalent`。数量和金额分别乘年化系数。

必须保留覆盖起止、覆盖天数/月当量、年化系数、实际量/金额、年化量/金额、方法及季节性风险。年化值只能称“12个月年化基线/比例外推”，不能写成历史实际全年值。

### Step 6 — SKU / Region Normalization

SKU优先 `物料编码 + 规格 + 单位`；无稳定编码时用 `名称 + 品牌 + 规格/型号 + 单位`。多区域按 `协议编号 + SKU + 区域` 聚合。不同规格、单位、税口径不得直接合并比价。

### Step 7 — Historical Spend Analysis for Strategy Report（强制尽可能完成）

该步骤同时服务需求预测与 `sourcing-strategy` 报告。

#### 7.1 Overall

至少输出历史有效期间、有效订单数/明细行数（可得）、实际采购金额、税口径、平均订单金额（可得）、部分周期的12个月年化金额。

#### 7.2 Region

存在区域/地址时输出订单数、金额、占比、Top 3 区域，以及主城/区县/跨城市或跨省结构（仅按真实数据）。

#### 7.3 Business Unit / 军种 / 业态 / 项目

存在真实业务维度字段时输出各维度订单数、金额、占比、Top 业务单元和集中度。不存在时记录 `dimension_unavailable`，不得从项目名或地址猜军种/业态。

#### 7.4 Order Amount Distribution

存在订单编号 + 订单金额时必须分析订单碎片化：先按订单编号汇总订单总金额，避免明细重复计数；至少形成4个可解释金额区间；优先沿用用户/企业已有区间，无既定区间时按数据分布设置并说明方法；输出各区间订单数、订单占比，可得时同时输出金额及金额占比，并识别小额高频/集中大单。

该结果用于下游“免运额度/最低起送/合单策略”，本 Skill 不决定正式阈值。

#### 7.5 SKU Spend / Pareto

输出 Top 5 高支出 SKU（不足5个全部）、金额/占比/数量/平均价（可得）、Top 5/Top 10 集中度（可算时）、累计达到80%支出的SKU数量和长尾结构。

#### 7.6 Supplier Concentration

存在供应商字段时输出 Top 3 历史供方金额/占比与集中度风险。历史供方分析不得改变当前官方邀标候选池。

#### 7.7 Price

对同品牌、同规格、同单位、同税口径可比 SKU 分析历史平均价、可得的最高/最低/中位/加权平均及时间变化。

#### 7.8 Analysis Conclusions

数据足够时，`strategy_report_handoff` 至少给出3条数字化关键支出发现和2条可供下游组织的采购含义，并记录缺失维度限制。不得只写一条整体采购金额。

### Step 8 — Annual Growth Rate Decision Gate（强制）

下一年度预计采购量 = `12个月年化基线 × (1 + 年度增量比例)`。

增量优先级：可比两年度数量同比 → 采购金额同比作为 `spend_yoy_proxy` → 企业正式业务规模/预算增幅 → 用户确认预估增量 → 用户确认0%。金额同比不等于价格增长，数量增长与价格增长必须分开。

缺少两年度可比数据时让用户选择：补充历史、提供预估增量、使用0%增量。确认前年度预计量仍填12个月年化基线，并标记 `provisional_annualized_baseline / pending_growth_confirmation`。

### Step 9 — Price & Amount Projection

有独立价格变化依据：`projected_unit_price = baseline_unit_price × (1 + price_growth_rate)`；`projected_amount = final_estimated_quantity × projected_unit_price`。无独立价格依据时内部金额测算可使用0价格增长并注明。

### Step 10 — Requirement List Output（固定）

使用 `templates/需求清单模板.xlsx` 生成 `{{项目名称}}-历史测算需求清单.xlsx`，年度预计采购量必须非空。可回填序号、区域、名称、品牌、规格、单位、起订量（有依据）、年度预计采购量、备注。供方报价字段 `含税单价/未税单价/税率/含税总价` 必须保持空白。

### Step 11 — Existing Requirement Conflict

原需求已有数量时不静默覆盖，输出原数量、年化基线、增量、建议量、差异和人工确认状态。

## 4. Structured Handoff

`requirement_handoff` 保持现有字段合同。

`strategy_report_handoff` 必须尽可能记录：selected_agreement_numbers、有效订单口径、覆盖周期/年化方法、原始与年化采购额、`spend_analysis`（整体 + 区域 + 业务单元 + 订单金额段 + SKU/Pareto + 历史供方集中度的可追溯摘要）、`price_analysis`、`key_findings`、`risk_flags`、`report_ready_facts`。

为了兼容当前 schema，多维结果写入 `strategy_report_handoff.spend_analysis` 的结构化文本条目，建议前缀：`[overall] / [region] / [business_unit] / [order_amount] / [sku] / [supplier]`。实际数字必须来自当前项目，不得复制示例。

## 5. Mandatory Audits

保留 annualization、growth_rate_decision、协议范围和订单过滤审计。

## 6. Guardrails

AI MUST NOT：未经用户选择默认协议；纳入无效订单；把部分周期当全年实际；自行制造增量；双重放大数量/价格；将年度预计量留空；将临时年化基线冒充最终需求；不同规格/单位/税口径直接合并；将历史价格填入供方报价字段；静默覆盖采购员数量；有可用支出维度时只给整体金额；虚构军种/业务单元；用历史供方排名改变邀标池；自行决定免运/最低起送正式阈值。

## 7. Human Review

采购员最终确认协议编号、无效订单过滤、SKU/区域映射、年化周期、增量处理、最终预计量/金额以及需正式落地的商务阈值。

## 8. Success Criteria

1. 用户明确选择协议；
2. 只有已完成/执行中订单参与；
3. 不足12个月生成12个月年化基线；
4. 年度预计采购量非空；
5. 增量缺数据时进入人工选择；
6. strategy_report_handoff 尽可能提供区域/业务单元/订单金额段/SKU/供方/价格多维结构；
7. 下游 sourcing-strategy 可直接形成数字化支出分析。
