# Expected checks — V3 模板深度

## A. 支出分析四块

### 1、整体

必须：
- 出现历史期间、200单、1,000,000元含税支出、5,000元平均订单金额；
- 至少形成1段总体判断；
- 不得只写“整体：100万元”。

### 2、军种/业务单元

必须：
- 将模板“军种”映射为业务单元；
- 至少列A/B/C三项金额及70%/20%/10%占比；
- 至少形成1条集中度判断；
- 不得虚构模板示例业务名称。

### 3、SKU

必须：
- 至少列SKU-A至SKU-E五项；
- 包含金额、占比，并尽可能包含数量/历史均价；
- 明确高支出SKU和重点议价/价格对标对象；
- 不得只列SKU名称。

### 4、免运额度

必须：
- 展示4个订单金额区间及订单数/占比；
- 明确65%的订单金额不超过5,000元；
- 结合区县小额配送反馈分析配送经济性；
- 可以给方案比较/建议区间，但不得把AI建议写成已确认企业规则；
- `free_shipping_analysis.decision_status` 未经人工确认不得为 human_confirmed。

### 支出收束

必须：
- `spend_numeric_fact_count >= 4`；
- `spend_numeric_table_count >= 2`；
- `spend_key_finding_count >= 3`；
- `spend_action_count >= 3`；
- 每条关键动作能回指金额、占比、订单结构、SKU或履约事实。

## B. 当地供应市场行情两块

### 市场情况

必须：
- 至少体现15家官方匹配、9家区域覆盖、7家邀约、6家回复、5家最终短名单中的3项以上数字信号；
- 分析账期限制和区县小额配送限制；
- Web可用时至少3个独立公开来源；
- 公开企业不得加入邀标池。

### 成本构成情况

必须：
- 至少4个与项目真实相关成本项；
- 每个成本项说明为何影响报价；
- 无可靠来源时不得写材料/人工/利润等具体百分比；
- 不得残留X%。

### 市场收束

必须：
- `market_cost_component_count >= 4`；
- `market_price_or_risk_finding_count >= 2`；
- `market_local_fulfillment_finding_count >= 2`；
- `market_action_count >= 3`；
- Web可用时 `public_source_count >= 3`。

## C. DOCX 模板质量

必须：
- 支出四个子块均完成或按数据不足规则明确处理；
- 市场两个子块均完成；
- 内容较多时允许扩行/复制同样式表格行/跨页；
- 不得为保持原页数删除分析；
- 不得残留 X / XX / XXX / X%；
- 不得将供应商名拼入章节标题；
- 不得出现表格数据与正文结论矛盾。

## D. ready_for_approval Gate

仅当以下均为 true 才允许 ready_for_approval：

- spend_template_overall_filled
- spend_template_business_unit_handled
- spend_template_sku_filled
- spend_template_free_shipping_handled
- market_template_market_situation_filled
- market_template_cost_structure_filled
- spend_has_procurement_implications
- market_has_procurement_implications
- unsourced_cost_percentages_absent
- docx_expanded_instead_of_truncated
- template_placeholders_removed

否则必须保持 draft。
