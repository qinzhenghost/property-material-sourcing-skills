# 协议编号选择规则

历史订单导出可能包含多个框架协议/合同。分析前必须识别全部唯一协议编号，并由用户明确选择一个或多个协议后才能继续。

可识别字段包括：协议编号、协议号、合同编号、合同号、框架协议编号、采购协议编号。

选择前需列出每个协议的原始订单数、有效订单数、有效明细行、数据期间、SKU数、有效含税金额和主要区域；有效订单仅指“已完成/执行中”。

即使只有一个协议，也必须用户确认；禁止默认选择最新、金额最大或全部协议。

用户多选时保留 `agreement_number` 维度，先分别统计再合并；同名SKU跨协议仍需按物料编码/规格/单位校验。

无协议编号订单默认排除，只有用户明确确认“包含未关联协议订单”后才能纳入。

审计至少记录：`available_agreements`、`selected_agreement_numbers`、`selection_confirmed`、`include_unlinked_agreement_records`、`unselected_order_count`、`unlinked_agreement_record_count`。
