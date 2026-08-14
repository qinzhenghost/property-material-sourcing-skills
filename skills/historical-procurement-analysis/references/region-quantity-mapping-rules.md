# Region Quantity Mapping Rules

## Purpose

当历史订单同时覆盖多个区域，而需求清单中存在同名、同品牌、同规格、同单位的重复 SKU 行时，定义历史采购量如何映射回需求清单。

## Core Rule

> 历史数量必须按需求清单真实业务维度映射，不得先跨区域汇总后再随意分配到重复行。

## Historical Region Source

优先使用官方订单导出中的显式字段：

1. `地址省份`
2. `地址城市`
3. `收货地址`
4. `使用部门 / 项目`

只做名称规范化，例如：

- `重庆` → `重庆市`
- `贵州省` → `贵州省`

不得根据供应商、项目名称或 AI 常识推断不存在的区域。

## Region-first Aggregation

历史采购聚合优先键：

`物料编码 + 规格 + 单位 + 区域`

如果没有稳定物料编码，再使用：

`物料名称 + 品牌 + 规格 + 单位 + 区域`

输出至少包括：

- historical_total_quantity
- quantity_by_region
- spend_by_region
- price_by_region（如有差异）

## Duplicate Requirement Lines Gate

若需求清单中存在：

- 同一名称；
- 同一品牌；
- 同一规格；
- 同一单位；
- 多条重复行；

且历史订单明确存在多个区域，则必须检查需求清单是否存在可用于区分的：

- 城市/区域；
- 项目；
- 收货范围；
- 其他明确业务维度。

### 如果有显式区域字段

可以执行：

`历史 SKU + 区域 → 对应需求行`

### 如果没有显式区域字段

结果必须是：

`requirement_region_mapping = unresolved`

并且：

- 不自动回填预计采购量；
- 输出每个区域历史数量供采购员确认；
- 要求在需求清单增加区域字段，或建立明确的行号→区域映射。

## Partial-period Guardrail

即使 SKU 与区域映射清晰，如果历史数据不是完整采购周期：

- 只能生成 `observed_historical_quantity`；
- 不得默认按天数/月数年化；
- 不得直接把样本量写成年度预计采购量。

只有在以下条件满足时才可生成年度建议数量：

1. 存在完整年度或可比同期基线；
2. 周期口径一致；
3. SKU 与区域均可比；
4. 下一周期业务规模变化有明确依据；
5. 采购员确认预测口径。

## Requirement Fill Formula

完整基线存在时：

`projected_quantity(region, sku) = baseline_quantity(region, sku) × (1 + confirmed_scale_growth_rate)`

如果存在 SKU 特殊调整：

`final_suggested_quantity = projected_quantity × (1 + confirmed_item_adjustment_rate)`

增长因子必须独立记录。

## Strategy Report Handoff

区域历史数据可用于采购策略报告：

- 区域采购金额结构；
- 区域 SKU 需求差异；
- 重点区域；
- 区域历史价格差异；
- 配送范围与需求规模说明。

但部分周期数据必须写明分析周期，不能表述成全年事实。

## Validation Case Learned from Real Sample

真实米面粮油历史订单验证中：

- 同一采购项目同时出现重庆、贵州采购记录；
- 同一 SKU 在两个区域存在不同历史数量；
- 现有需求清单部分 SKU 使用重复行，但没有显式城市/区域列。

因此该场景必须触发 `Duplicate Requirement Lines Gate`，禁止 AI 直接将历史数量静默写入重复行。
