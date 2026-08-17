# 12个月年化与年度增量规则

## 1. 目的

历史订单可能只覆盖数月，但需求清单必须给出“年度预计采购量”。本规则把两个问题分开：年化基线、年度增量。

## 2. 年化基线

完整12个月：`annualization_factor = 1`，`annualized_baseline_quantity = actual_quantity`。

不足12个月：

- 完整自然月：`annualization_factor = 12 / covered_months`
- 首尾月不完整/订单明细日期：`covered_month_equivalent = covered_days / 365.2425 × 12`
- `annualization_factor = 12 / covered_month_equivalent`
- `annualized_baseline_quantity = observed_quantity × annualization_factor`
- `annualized_baseline_amount = observed_amount × annualization_factor`

年化值必须标记为比例外推，不得写成历史全年实际。

## 3. 年度增量比例

优先使用：可比两年度数量同比；其次可比两年度采购金额同比 `spend_yoy_proxy`；再其次企业正式业务增幅、用户确认预估增量比例、用户确认0%增量。

两年度采购金额同比：

`spend_yoy_rate = (later_year_spend - earlier_year_spend) / earlier_year_spend`

金额同比可能包含价格变化，不得同时作为数量增长和价格增长。

## 4. 缺少两年度可比数据

必须让用户选择：

1. 补充另一年度同口径数据；
2. 提供预估增量比例；
3. 按0%增量。

用户未确认前，“年度预计采购量”仍不可为空：先填12个月年化基线，并标记 `pending_growth_confirmation`。

用户确认后：

`final_estimated_quantity = annualized_baseline_quantity × (1 + confirmed_growth_rate)`
