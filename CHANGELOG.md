# CHANGELOG

## v0.9.1

强化 `sourcing-strategy` 报告内容深度，重点解决测试中“支出分析过薄、当地供应市场行情分析仅保留提纲/占位”的问题。

- `sourcing-strategy` 内容规则升级至 `0.7.2`；
- 支出分析新增强制最低深度：整体实际/年化口径、区域/项目/业态 Top 维度、Top 5 SKU、历史供方集中度、可比价格、数字化关键发现及采购含义；
- 无完整历史数据时不得只写“暂无数据”，必须降级做计划支出/数量结构分析并明确数据缺口；
- 当地供应市场行情改为“内部供应市场信号 + 外部公开市场研究 + 采购策略影响”三层结构；
- Web 可用且地区/品类明确时必须主动研究，原则上至少3个独立公开来源并优先近12个月资料；
- 无可靠证据时禁止填写材料/人工/利润等成本百分比；
- 新增 `report-content-quality-guide.md`，统一约束需求背景、支出、市场、短名单、供应商选择、目标价、定标、备选机制和风险章节写法；
- 短名单章节要求先说明官方库→符合条件→邀约→回复→最终入围漏斗，再逐家写可追溯入围理由；
- 风险章节优先采用“触发场景 → 证据/条件 → 影响 → 处置 → 是否需人工确认”；
- `sourcing-strategy.schema.yaml` 增加 spend/market completeness status、维度 breakdown、关键发现、采购含义及完整性 consistency checks；
- 新增 `thin-analysis-regression` 回归测试；
- 最终 DOCX 禁止残留 `X / XX / XXX / X%`、空分析标题和章节字段错位；
- 本轮参考企业采购方案案例仅提炼通用内容结构，未将真实项目名称、供应商、金额、人员或时间提交到仓库。

## v0.9.0

结构性合并：`shortlist-approval` → `supplier-shortlist`。

- `supplier-shortlist` 升级至 `0.5.0`；
- 原供方回复汇总与短名单草稿流程保留为 Phase A；
- Phase A 继续使用企业 `供方短名单模板.xlsx`，输出 `{{项目名称}}-供方短名单.xlsx`；
- AI 仅可输出 `建议入围 / 待澄清 / 不建议入围`，不得将建议写成最终短名单；
- Phase A 结束后强制暂停等待采购员人工确认最终入围 / 不入围 / 待澄清状态；
- 原 `shortlist-approval` 流程吸收到 Phase B；
- Phase B 只能消费采购员确认结果，不得重新评分、重新排序或改变短名单结论；
- Phase B 固定输出最终 `{{项目名称}}-供方短名单.xlsx`、`{{项目名称}}-短名单报批邮件.md`、`{{项目名称}}-shortlist-handoff.yaml`；
- 原 `shortlist-approval-email.md` 模板迁入 `supplier-shortlist/templates/`；
- 新增 `schemas/supplier-shortlist.schema.yaml`，统一承载 AI 建议、人工确认、报批邮件与 strategy_handoff；
- 删除旧 `schemas/shortlist-approval.schema.yaml`；
- 删除独立 `shortlist-approval` Skill canonical / 兼容镜像入口；
- `sourcing-strategy` 升级至 `0.7.1`，Required Input 改为 `supplier-shortlist.strategy_handoff`；
- sourcing-strategy 的短名单一致性检查改为只接受人工确认后的 shortlist_handoff；
- sourcing-strategy 的保证金优先消费 `sourcing-invitation.bid_bond` 当前项目可追溯计算结果；
- 主流程由 6 个 Skill 压缩为 5 个 Skill；
- `.agents/skills/` 与兼容 `skills/` 同步。

## v0.8.0

结构性合并：`official-supplier-matching` → `sourcing-invitation`。

- `sourcing-invitation` 升级至 `0.4.0`；
- 原官方供方匹配流程完整吸收到 `sourcing-invitation` Phase A；
- 保留 official registry only、registry tier、物资类 Gate、品类 Gate、区域 Gate、资质/状态 Gate、Fit、evidence、候选池分组和官方库覆盖缺口；
- Phase A 候选池生成后必须暂停等待采购员人工确认初版供方；
- 人工确认后同一个 Skill 进入 Phase B 邀标准备；
- Phase B 固定生成四件套：单一 BCC 邀标邮件 `.eml`、最终需求清单 `.xlsx`、企业原版招标意向征集登记表 `.xlsx`、采购内部供方信息长名单 `.xlsx`；
- 04 长名单只覆盖人工确认初版供方，联系人/电话/邮箱仅来自官方库或采购员确认信息；
- BCC 邮箱必须能回溯到 04 长名单；
- `material-requirement-analysis` 升级至 `0.1.2`，下游改为 `sourcing-invitation / official_supplier_matching phase`；
- `sourcing-invitation-package.schema.yaml` 升级至 `0.4.0`，统一承载官方供方匹配、人工供方确认、联系人长名单、BCC、保证金和四件套状态；
- 原 `supplier-match.schema.yaml` 不再作为独立 Skill Handoff Schema；
- 原 `official-supplier-matching` Skill 入口与其旧目录删除；
- 原官方供方来源政策和品类/区域匹配规则迁移至 `sourcing-invitation/references/`；
- `.agents/skills/` 与兼容 `skills/` 同步。

## v0.7.8

`sourcing-invitation` 升级为 `0.3.3`：

- 企业《招标意向征集登记表模板.xlsx》`Sheet1!N3` 中原固定“预计6000元投标保证金”改为 `{{投标保证金金额}}` 变量；
- 投标保证金计算口径固定为 `采购清单预估总金额 × 1%`；
- 计算结果使用 `CEILING(..., 1000)` 向上取整到 1000 元整数倍；
- 采购清单预估总金额必须来自当前项目可追溯的 confirmed requirement、historical/material requirement handoff 或采购员明确确认；
- 禁止从供应商待填写报价列反推预估总金额；
- 禁止沿用历史模板的 6000 元或其他默认保证金；
- 缺少采购清单预估总金额时，03 登记表不得标记为 ready；
- 新增 `references/bid-bond-variable-rules.md`；
- `sourcing-invitation-package.schema.yaml` 升级至 `0.3.3`；
- `.agents/skills/` 与兼容 `skills/` 的登记表模板同步更新为 N3 变量版。

## v0.7.7

`sourcing-invitation` 升级为 `0.3.2`：

- 用户此前提供的企业《招标意向征集登记表.xlsx》作为唯一正式模板；
- 仓库模板替换为原始附件对应版本；
- 新增 Original Template Integrity Gate；
- 03 交付物必须通过“复制企业原模板 → 仅填当前项目允许字段 → 另存为项目文件”生成；
- 禁止重新设计、重建、调整版式或使用通用表格替代企业模板；
- 必须保留工作表、行列结构、合并单元格、样式、打印设置、公式/验证和供方填写区域；
- `sourcing-invitation-package.schema.yaml` 升级至 `0.3.2`。

## v0.7.6

`sourcing-invitation` 升级为 `0.3.1`：

- 邮件交付物由 Markdown 正文升级为单一 `.eml` 草稿；
- 每个项目只生成一个 EML；
- 人工确认且邮箱已确认的供方统一写入 `Bcc`；
- 供方邮箱不得出现在 `To` 或 `Cc`；
- 缺失、来源不明确或冲突的供方邮箱不得猜测；
- EML 不嵌入任何 Excel 附件；
- 两个 Excel 独立交付；
- 新增 `references/eml-delivery-rules.md`；
- `sourcing-invitation-package.schema.yaml` 升级至 `0.3.1`。

## v0.7.5

`material-requirement-analysis` 升级为 `0.1.1`：

- P0 阻断项全部解除后，需求清单 Excel 变为强制交付物；
- 若仍有 P1/P2，输出 `{{项目名称}}-澄清版需求清单.xlsx`；
- 关键需求及项目商务条件均确认后，输出 `{{项目名称}}-最终需求清单.xlsx`；
- P1/P2 非阻断事项必须留痕，但不得阻止 Excel 生成；
- 输入本身为 Excel 时优先复制原文件并保留企业版式；
- 新增标准需求清单模板与输出规则；
- 供方报价列保持空白。

## v0.7.4

`historical-procurement-analysis` 升级为 `0.5.5`：

- 有效历史数据不足12个月时必须折算为12个月采购基线；
- 完整自然月使用 `12 / covered_months`，首尾月不完整时按日期月当量；
- 新增 annualization / growth_rate_decision；
- 年度预计采购量始终非空；
- 无两年度可比数据时提示补数据 / 给预估增量 / 使用0%增量；
- 用户确认增量后按年化基线乘以增长率更新最终年度预计采购量。

## v0.7.3

`historical-procurement-analysis` 升级为 `0.5.4`：

- 新增强制 Agreement Selection Gate；
- 自动识别协议/合同编号字段；
- 用户必须明确选择一个或多个协议编号；
- 禁止默认选择最新、金额最大或全部协议；
- 多协议分析保留 agreement_number 维度；
- 无协议编号订单默认排除。

## v0.7.2

`historical-procurement-analysis` 规则增强：

- 仅允许 `已完成`、`执行中` 订单进入统计和预测；
- 其他状态全部排除；
- 无订单状态字段时禁止静默使用全部数据；
- 新增过滤审计和需求清单固定输出；
- 新增区域列模板。

## v0.7.1

- 新增 `.agents/skills/` 作为 Codex 仓库级 Skill 主运行入口；
- 7 个现有子 Skill 映射到 `.agents/skills/`；
- `skills/` 暂保留兼容。

## v0.7.0

- 新增 `sourcing-strategy`；
- 对接企业《物资采购方案报告模板.docx》；
- 汇总历史采购、最终需求、官方供方、短名单和采购规则；
- 增加一致性检查。

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
