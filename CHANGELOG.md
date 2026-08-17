# CHANGELOG

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
