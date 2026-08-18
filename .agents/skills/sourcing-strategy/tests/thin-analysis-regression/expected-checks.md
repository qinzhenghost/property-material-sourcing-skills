# Expected checks

## A. 支出分析

必须通过：

- 明确写出历史数据覆盖期；
- 同时出现 600,000 元历史实际支出与 1,200,000 元12个月年化基线，并区分实际值/年化值；
- 体现120单及平均订单金额5,000元；
- 至少列出3个区域金额、占比和订单数；
- 至少列出3个业务单元金额、占比和订单数；
- 必须分析4个订单金额区间，并指出3,000元以下订单占35%；
- 至少列出5个SKU金额与占比；
- 至少列出3个历史供方支出及占比；
- 应识别 SKU-A 为最高支出SKU，历史供方A为最高支出供方；
- 至少3条带数字证据的关键发现；
- 至少2条直接对应本项目的采购策略含义；
- “免运额度/最低配送金额”必须引用订单金额结构、区域履约或供方最低起送反馈作为依据；
- 未经采购员确认，不得把AI建议的免运/最低起送阈值写成正式合同规则；
- 不得把6个月60万元直接表述为全年历史实际支出；
- 不得只输出“整体 / 军种维度 / SKU维度 / 免运额度”等标题。

## B. 当地供应市场行情分析

必须通过：

- 内部供应信号至少体现：12家官方匹配、8家明确覆盖、6家邀约、5家回复、4家短名单；
- 应分析120天账期接受风险；
- 应分析零散配送最低起送要求对报价/履约的影响；
- Web可用时至少3个独立公开来源；
- 外部分析应覆盖可获得的供应格局、上游/成本驱动、物流履约、价格趋势、政策/标准；
- 成本构成有比例时必须有明确来源；无可靠来源时只能定性描述，不得虚构百分比；
- 至少3条与项目直接相关的采购动作；
- 公开市场企业不得被加入邀标候选池或短名单；
- 不得只写“市场成熟、竞争充分，建议择优采购”等泛化句子。

## C. strategy-data.yaml

必须通过：

- `schema_version = 0.7.3`；
- `spend_analysis.completeness_status` 存在且取值合法；
- `spend_analysis.dimension_coverage` 明确记录 region / business_unit / order_amount / sku / supplier / price 可用性与是否分析；
- `spend_analysis.order_amount_distribution` 能承载订单金额区间；
- `spend_analysis.free_shipping_analysis.decision_status = pending_human_confirmation`，除非测试明确给出采购员确认值；
- `market_analysis.internal_supply_signals` 非空；
- `market_analysis.cost_drivers` 非空；
- `market_analysis.local_fulfillment_factors` 非空；
- `market_analysis.sourcing_implications` 至少3条；
- `content_depth_checks.spend_has_numeric_structure = true`；
- `content_depth_checks.spend_available_dimensions_analyzed = true`；
- `content_depth_checks.spend_has_procurement_implications = true`；
- `content_depth_checks.free_shipping_supported_by_order_or_fulfillment_evidence = true`；
- `content_depth_checks.market_has_internal_supply_signals = true`；
- `content_depth_checks.market_has_cost_drivers = true`；
- `content_depth_checks.market_has_local_fulfillment_view = true`；
- `content_depth_checks.market_has_procurement_implications = true`；
- `content_depth_checks.unsourced_cost_percentages_absent = true`；
- `content_depth_checks.template_placeholders_removed = true`。

## D. DOCX 模板与完整度

必须通过：

- DOCX 基于企业模板生成；
- 允许分析表格自然扩展/跨页，不得因维持原页数删减核心分析；
- DOCX 不残留 `X / XX / XXX / X%`；
- 不存在空的“市场情况 / 成本构成情况 / 军种维度 / SKU维度 / 免运额度”等标题；
- 供应商名称不得拼接到“当地供应市场行情分析”等章节标题；
- 支出数据表后必须有分析判断和采购含义；
- 市场章节必须有市场情况、成本驱动、价格/履约因素和采购策略影响；
- 模板或参考案例中的金额、供应商、日期、人员、成本比例不得无来源继承。

任一关键项失败，则本测试应判 FAIL，不得把内容过薄的报告视为成功。
