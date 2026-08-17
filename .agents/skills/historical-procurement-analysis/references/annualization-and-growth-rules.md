# 12个月年化与年度增量规则

## 1. 目的

历史订单可能只覆盖数月，但需求清单必须给出“年度预计采购量”。

本规则把两个问题分开：

1. **年化基线**：把不足12个月的有效历史采购量按比例折算到12个月；
2. **年度增量**：在年化基线上叠加可追溯的年度增长/下降比例。

## 2. 年化基线

### 完整年度

完整12个月：

`annualization_factor = 1`

`annualized_baseline_quantity = actual_quantity`

### 不足12个月

优先使用实际覆盖周期。

若数据按完整自然月提供：

`annualization_factor = 12 / covered_months`

`annualized_baseline_quantity = observed_quantity × annualization_factor`

若首尾月不完整或为订单明细日期：

`covered_month_equivalent = covered_days / 365.2425 × 12`

`annualization_factor = 12 / covered_month_equivalent`

`annualized_baseline_quantity = observed_quantity × annualization_factor`

金额同理：

`annualized_baseline_amount = observed_amount × annualization_factor`

## 3. 年化必须标明风险

按比例年化可能忽略季节性、节假日集中采购、一次性项目、协议上线/切换以及项目新增/退出。

因此年化值必须标记为 `calculated`，不能写成历史全年实际。

如果覆盖不足1个月或数据明显集中，仍生成12个月基线，但可信度应为 `low`。

## 4. 年度增量比例

优先级：

1. 可比两年度 SKU 数量同比；
2. 可比两年度采购金额同比，作为 `spend_yoy_proxy`；
3. 企业正式业务规模/预算增幅；
4. 用户确认的预估增量比例；
5. 用户确认0%增量。

两年度采购金额同比：

`spend_yoy_rate = (later_year_spend - earlier_year_spend) / earlier_year_spend`

金额同比可能包含价格变化，因此不得同时把该比例当数量增长和价格增长；如价格同比变化明显，必须提示。

## 5. 缺少两年度可比数据

必须提醒用户选择：

### A. 补充数据
补充另一年度同口径订单/年度采购金额。

### B. 提供预估增量比例
例如 `+5%`、`+8%`、`-3%`。

### C. 使用0%增量
直接采用12个月年化基线。

## 6. 用户未确认增量时

需求清单“年度预计采购量”仍然不能留空。

先写 `annualized_baseline_quantity`，并标记：

- `quantity_status = provisional_annualized_baseline`
- `human_confirmation_status = pending_growth_confirmation`
- 备注：“当前为12个月年化基线，年度增量比例待确认。”

用户确认增量后：

`final_estimated_quantity = annualized_baseline_quantity × (1 + confirmed_growth_rate)`

并更新为 `final_projected`。

## 7. 例子

历史有效订单覆盖6个月，某SKU采购量1000桶：

`12个月年化基线 = 1000 × 12 / 6 = 2000桶`

如果两年度数据不足，先输出2000桶并询问用户：补充数据、提供增量比例，或按0%增量。

若用户确认增量8%：

`最终年度预计采购量 = 2000 × 1.08 = 2160桶`
