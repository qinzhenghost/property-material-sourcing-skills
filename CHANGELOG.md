# CHANGELOG

## v0.7.7

`sourcing-invitation` 升级为 `0.3.2`：

- 确认用户此前提供的企业《招标意向征集登记表.xlsx》应作为唯一正式模板；
- 仓库 `.agents/skills/sourcing-invitation/templates/招标意向征集登记表模板.xlsx` 已替换为该原始附件的字节级一致文件；
- 兼容目录 `skills/sourcing-invitation/templates/` 同步使用同一原始模板；
- 新增 Original Template Integrity Gate；
- 03 交付物必须通过“复制企业原模板 → 仅填当前项目允许字段 → 另存为项目文件”生成；
- 禁止重新设计、重建、调整版式或使用通用表格替代企业模板；
- 必须保留工作表、行列结构、合并单元格、样式、打印设置、公式/验证和供方填写区域；
- 无法保证模板完整性时必须 BLOCK，不得交付替代版；
- `sourcing-invitation-package.schema.yaml` 升级至 `0.3.2`，新增 `template_source_mode`、`template_integrity_preserved` 与原模板一致性检查字段。

## v0.7.6

`sourcing-invitation` 升级为 `0.3.1`：

- 邮件正式交付物由 Markdown 正文升级为单一 `.eml` 草稿；
- 每个项目只生成一个 EML，不为每家供方分别生成；
- 人工确认且邮箱已确认的供方统一写入 `Bcc`；
- 供方邮箱不得出现在 `To` 或 `Cc`；
- `To` 仅可使用采购方明确提供的本方发送/归档邮箱，否则留空待人工确认；
- `Cc` 仅可使用采购员明确提供的内部抄送邮箱；
- 缺失、来源不明确或冲突的供方邮箱不得猜测，必须进入人工补充/确认；
- EML 不嵌入任何 Excel 附件，`attachments_embedded = false`；
- 邮件正文只引用最终需求清单和招标意向征集登记表的文件名；
- `{{项目名称}}-最终需求清单.xlsx` 与 `{{项目名称}}-招标意向征集登记表.xlsx` 继续作为两个独立文件交付；
- 实际发送前由采购员人工将两个 Excel 添加为附件；
- 新增 `references/eml-delivery-rules.md`；
- `sourcing-invitation-package.schema.yaml` 升级至 `0.3.1`。

## v0.7.5

`material-requirement-analysis` 升级为 `0.1.1`：

- P0 阻断项全部解除后，需求清单 Excel 变为强制交付物；
- 若仍有 P1/P2，输出 `{{项目名称}}-澄清版需求清单.xlsx`；
- 关键需求及项目商务条件均确认后，输出 `{{项目名称}}-最终需求清单.xlsx`；
- P1/P2 非阻断事项必须留痕，但不得阻止 Excel 生成；
- 输入本身为 Excel 时优先复制原文件并保留企业版式，不覆盖原始文件；
- 无可复用模板时使用 `templates/标准需求清单模板.xlsx`；
- 新增 `references/requirement-workbook-output-rules.md`；
- 供方报价列（含税单价/未税单价/税率/含税总价）保持空白，不得写入历史价、旧报价或 AI 测算价；
- P0 清零后若只有文字/YAML/JSON而没有 Excel，则 Skill 执行状态视为 `incomplete`；
- README 与 Codex 测试示例同步更新。

## v0.7.4

`historical-procurement-analysis` 升级为 `0.5.5`：

- 将原“部分周期不年化”规则调整为“有效历史数据不足12个月时必须折算为12个月采购基线”；
- 完整自然月使用 `12 / covered_months` 折算；首尾月不完整时按有效数据起止日期换算月当量；
- 新增 `annualization` 结构：覆盖日期、覆盖天数/月数、月当量、年化系数、季节性风险；
- 年化值明确标记为比例外推，不得写成历史全年实际；
- 需求清单 `年度预计采购量` 不再允许留空；
- 当年度增量比例尚未确认时，先填12个月年化基线，并标记 `provisional_annualized_baseline` / `pending_growth_confirmation`；
- 新增 `growth_rate_decision`；
- 年度增量优先使用可比两年度数量同比，其次可比两年度采购金额同比 `spend_yoy_proxy`、正式业务规模增幅或用户确认的预估增量；
- 缺少两年度可比数据时必须提示用户三选一：补充另一年度数据 / 提供预估增量比例 / 使用0%增量；
- 用户确认增量后按 `annualized_baseline_quantity × (1 + confirmed_growth_rate)` 更新最终年度预计采购量；
- 采购金额同比不得同时作为数量增长和价格增长，防止重复放大；
- 新增 `references/annualization-and-growth-rules.md`；
- `historical-procurement.schema.yaml` 与示例升级至 `0.5.5`。

## v0.7.3

`historical-procurement-analysis` 升级为 `0.5.4`：

- 新增强制 `Agreement Selection Gate`；
- 自动识别 `协议编号 / 协议号 / 合同编号 / 合同号 / 框架协议编号 / 采购协议编号` 等字段；
- 先列出全部唯一协议及其原始订单数、有效订单数、有效明细行、数据期间、SKU数、有效采购金额和区域；
- 用户必须明确选择一个或多个协议编号后才能进入正式历史分析；
- 即使只有一个协议编号，也不能自动跳过确认；
- 禁止默认选择最新协议、金额最大协议或全部协议；
- 多协议分析时保留 `agreement_number` 维度，先分别统计再合并；
- 无协议编号订单默认排除，仅在用户明确确认后允许纳入；
- `requirement_handoff` 和 `strategy_report_handoff` 增加所选协议范围；
- `historical-procurement.schema.yaml` 升级至 `0.5.4`；
- 新增 `references/agreement-selection-rules.md`；
- 重写主 `SKILL.md` 为标准 UTF-8，修复历史 Git Blob 编码异常。

## v0.7.2

`historical-procurement-analysis` 规则增强：

- Skill 版本升级为 `0.5.3`；
- 历史订单统计前新增强制 `Order Status Filter Gate`；
- 只允许 `已完成`、`执行中` 进入历史采购量/金额/价格/同比/预测；
- `已取消`、`已退货` 及其他状态全部排除；
- 无订单状态字段时禁止静默使用全部数据，除非采购员确认源数据已预过滤；
- 新增过滤审计：原始行数、保留/剔除行数、按状态剔除数量；
- 新增 `order-status-filter-and-requirement-output.md`；
- 新增 `templates/需求清单模板.xlsx`，显式加入“区域”列；
- 新增固定输出 `{{项目名称}}-历史测算需求清单.xlsx`；
- 历史分析只回填采购方需求字段，含税单价/未税单价/税率/含税总价保持空白；
- `historical-procurement.schema.yaml` 升级至 0.5.3；
- 更新历史采购 Handoff 示例。

## v0.7.1

- 新增 `.agents/skills/`，作为 Codex 仓库级 Skill 主运行入口；
- 7 个现有子 Skill 完整映射到 `.agents/skills/`；
- `skills/` 暂保留兼容；
- README 增加 Codex 单 Skill 测试说明。

## v0.7.0

- 新增 `sourcing-strategy`；
- 对接企业《物资采购方案报告模板.docx》；
- 汇总历史采购、最终需求、官方供方、短名单和采购规则；
- 增加数量/预算/区域/短名单/商务条件/定标规则一致性检查。

## v0.6.0

- 新增 `shortlist-approval`；
- 固定输出短名单报批邮件草稿 + strategy_handoff。

## v0.5.2

- 历史采购量扩展为 `SKU + 区域` 聚合；
- 同 SKU 重复需求行未标区域时禁止自动回填。

## v0.5.1

- 基于真实订单导出固定历史订单字段映射；
- 部分周期数据不得机械年化。

## v0.5.0

- 新增 `historical-procurement-analysis`。

## v0.4.0

- 新增 `supplier-shortlist`。

## v0.3.0

- 新增 `sourcing-invitation` 及邀标三件套。

## v0.2.0

- 新增 `official-supplier-matching`，明确官方供方库 ONLY。

## v0.1.0

- 新增 `material-requirement-analysis`、requirement/supplier schemas 和米面粮油需求诊断案例。
