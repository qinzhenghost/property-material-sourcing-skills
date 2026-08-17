# 订单导出字段映射

历史采购订单标准字段至少包括：

| 原始字段/同义字段 | 标准字段 | 用途 |
|---|---|---|
| 协议编号 / 协议号 / 合同编号 / 合同号 / 框架协议编号 / 采购协议编号 | `agreement_number` | 用户明确选择分析协议范围 |
| 订单编号 | `order_id` | 订单去重 |
| 订单状态 | `order_status` | 仅保留已完成/执行中 |
| 订单创建日期 | `order_date` | 周期识别 |
| 物料编码 | `material_code` | SKU稳定标识 |
| 物料名称 | `material_name` | SKU识别 |
| 物料规格 | `specification` | SKU可比性 |
| 品牌 | `brand` | SKU可比性 |
| 数量 | `quantity` | 历史采购量 |
| 单位 | `unit` | 单位一致性 |
| 含税单价 | `tax_included_unit_price` | 历史价格基线 |
| 含税总额 | `tax_included_amount` | 支出分析 |
| 不含税总额 | `pre_tax_amount` | 支出分析 |
| 供应商名称 | `supplier_name` | 历史合作/集中度 |
| 收货地址 / 地址省份 / 地址城市 | `delivery_location / province / city` | 区域分析 |

处理顺序必须为：`协议识别 → 用户选择协议 → 订单状态过滤 → 周期完整性判断 → SKU+区域分析`。

协议字段缺失时不得默认整个文件属于同一协议；无协议编号记录默认排除。

SKU优先聚合键：`agreement_number + material_code + specification + unit + region`。无物料编码时使用名称+品牌+规格+单位，并保留协议与区域维度。

部分周期数据不得机械年化；历史订单中的敏感联系人、电话、地址等不得进入公开示例。
