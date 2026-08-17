# 协议编号选择规则

## 目的

历史订单导出可能同时包含多个框架协议、合同或采购协议。不同协议可能存在不同采购周期、SKU范围、区域、价格和供应商，因此不能默认把整个订单导出作为同一个历史采购基线。

## 1. 协议字段识别

优先识别以下字段：

- 协议编号
- 协议号
- 合同编号
- 合同号
- 框架协议编号
- 采购协议编号

不得根据项目名称、供应商名称或文件名猜测协议编号。

## 2. 用户选择前的概览

对全部唯一协议编号先生成：

| 协议编号 | 原始订单数 | 有效订单数 | 有效明细行 | 数据期间 | SKU数 | 有效含税金额 | 主要区域 |
|---|---:|---:|---:|---|---:|---:|---|

其中有效订单仅指状态为：

- 已完成
- 执行中

同时单独列出：

- 无协议编号记录数；
- 已取消/已退货等排除订单数；
- 无法识别状态的订单数。

## 3. 必须由用户明确选择

即使只识别到一个协议编号，也必须由用户确认。

示例：

```text
选择协议：
- HT-2025-001
```

或者：

```text
选择协议：
- HT-2025-001
- HT-2025-003
```

在用户确认前：

- 不形成正式历史基线；
- 不生成预测数量；
- 不向需求清单写入年度预计采购量；
- 不进入后续 Skill。

禁止默认选择最新协议、金额最大协议或全部协议。

## 4. 多协议分析

用户多选协议后：

1. 保留 `agreement_number` 维度；
2. 每个协议先独立统计；
3. 再输出所选协议合并统计；
4. 同名SKU跨协议仍需按物料编码/规格/单位验证可比性；
5. 不同协议价格不可无条件混成一个简单平均价。

推荐稳定键：

`agreement_number + material_code + specification + unit + region`

## 5. 无协议编号订单

无协议编号的订单默认排除，并单独汇报行数、订单数和金额（如可计算）。

只有用户明确要求并确认“包含未关联协议订单”后才能纳入，同时 Handoff 必须记录该例外。

## 6. 审计结构

```yaml
agreement_selection:
  agreement_field: 协议编号
  available_agreements:
    - agreement_number: HT-2025-001
      raw_order_count: 100
      valid_order_count: 80
      valid_record_count: 120
      date_from: 2025-01-01
      date_to: 2025-12-31
      sku_count: 30
      valid_tax_included_amount: 1000000
      regions: [重庆市]
  selected_agreement_numbers:
    - HT-2025-001
  selection_confirmed: true
  include_unlinked_agreement_records: false
  unselected_order_count: 20
  unlinked_agreement_record_count: 0
```
