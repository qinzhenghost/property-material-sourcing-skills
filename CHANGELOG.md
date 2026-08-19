# CHANGELOG

## v0.11.0

目录收敛与模板完整性修复。

- 删除根目录 `skills/` 兼容镜像；
- `.agents/skills/` 成为仓库唯一 Skill Source of Truth；
- README / AGENTS 移除双目录维护说明；
- 保留统一主入口 `.agents/skills/property-material-sourcing/SKILL.md` 与 5 个内部专业模块；
- 企业 Office 模板继续只保存在对应 `.agents/skills/<module>/templates/` 下，不做重复副本；
- 检查发现 GitHub 中 `.agents/skills/sourcing-invitation/templates/供方信息长名单模板.xlsx` ZIP/OOXML 包损坏：原文件 Blob `4ff7653fbcdadbb314bab505beb08f87c8ea756a`、大小 5282 bytes，ZIP 检查提示缺失数据；
- 已使用本地验证通过的无“供方编码”8列版本替换，Blob 更新为 `e894c1f1ac5a1a556d3cab2dbb2f2f5fd6b118ba`、大小 5684 bytes；
- 修复后的供方长名单字段固定为：序号、供方名称、联系人姓名、联系电话、邮箱地址、联系信息来源、联系信息状态、备注；
- 修复版通过 ZIP 完整性、OOXML 核心结构、`artifact_tool` 实际导入和公式错误扫描。

## v0.10.0

将现有 5 个物业物资采购 Skill 整合为一个统一推荐入口。

- 新增 `.agents/skills/property-material-sourcing/SKILL.md`，版本 `0.1.0`；
- 新增兼容镜像 `skills/property-material-sourcing/`，canonical / mirror 主规则保持字节一致；
- 原 5 个 Skill 保留为内部专业模块，避免模板和成熟规则在整合过程中被破坏；
- 统一编排 Phase 0 历史采购分析（可选）→ Phase 1 需求分析 → Phase 2 官方供方匹配与邀标 → waiting supplier replies → Phase 3 短名单与报批 → Phase 4 采购策略；
- 新增 `schemas/property-material-sourcing-workflow.schema.yaml`，管理当前阶段、Human checkpoint、blocker、artifact registry、resume state 和 module versions；
- 新增断点续跑规则：用户说“继续”时从最新已验证 checkpoint 继续，不重复未变化的人工确认阶段；
- 上游事实发生实质变化时，下游旧产物标记 `superseded` 并明确重跑范围；
- 保留协议范围、年度增量、P0、初版邀标供方、人工发送、最终短名单、预算/定标等全部 Human Gate；
- Phase 2 继续固定交付单一 BCC EML + 最终需求清单 + 企业招标意向征集登记表 + 内部供方信息长名单；
- Phase 3 继续严格分离 AI 短名单建议与采购员最终确认；
- Phase 4 继续使用 `sourcing-strategy 0.7.4` 的 V3 深度支出分析和当地供应市场行情 Gate；
- 新增统一 full-flow、no-history、resume 回归检查；
- 新增根级 `AGENTS.md`，要求端到端任务优先通过统一主 Skill 路由，五个原 Skill 仅作为内部模块或单阶段调试入口；
- 企业 Excel/DOCX 二进制模板不搬迁、不重建，继续由原内部模块目录管理。

## v0.9.1

强化 `sourcing-strategy` 报告内容深度，重点解决测试中“支出分析过薄、当地供应市场行情分析仅保留提纲/占位”的问题。

- 强化支出分析：整体、区域/项目/业态、订单金额段、Top SKU、历史供方、价格、免运/最低配送依据；
- 无完整历史数据时不得只写“暂无数据”，必须降级做计划支出/数量结构分析并明确数据缺口；
- 当地供应市场行情改为“内部供应市场信号 + 外部公开市场研究 + 本地履约 + 采购策略影响”；
- Web 可用且地区/品类明确时必须主动研究，公开市场信息不得新增邀标候选供方；
- 无可靠证据时禁止填写材料/人工/利润等成本百分比；
- 新增 `report-content-quality-guide.md`、`template-v3-content-blueprint.md` 等内容规则；
- 新增/强化 `thin-analysis-regression` 与 V3 模板深度回归；
- 最终 DOCX 禁止残留 `X / XX / XXX / X%`、空分析标题和章节字段错位；
- `historical-procurement-analysis` 升级到 `0.5.6`，为策略报告提供更丰富的支出 Handoff；
- `sourcing-strategy` 后续强化到 `0.7.4`，加入模板四块支出分析、市场两块内容及机器可检查的内容深度指标。

## v0.9.0

结构性合并：`shortlist-approval` → `supplier-shortlist`。

- `supplier-shortlist` 升级至 `0.5.0`；
- 原供方回复汇总与短名单草稿流程保留为 Phase A；
- AI 仅可输出 `建议入围 / 待澄清 / 不建议入围`，不得将建议写成最终短名单；
- Phase A 结束后强制暂停等待采购员人工确认；
- 原 `shortlist-approval` 流程吸收到 Phase B；
- Phase B 只能消费采购员确认结果，不得重新评分、重新排序或改变短名单结论；
- 固定输出最终短名单、短名单报批邮件、shortlist-handoff；
- 新增统一 `schemas/supplier-shortlist.schema.yaml`；
- 删除独立 `shortlist-approval` 入口；
- `sourcing-strategy` 只消费人工确认 shortlist_handoff；
- 主流程由 6 个 Skill 压缩为 5 个 Skill。

## v0.8.0

结构性合并：`official-supplier-matching` → `sourcing-invitation`。

- `sourcing-invitation` 升级至 `0.4.0`；
- 原官方供方匹配完整吸收到 Phase A；
- 保留 official registry only、品类/区域/资质/状态 Hard Gate、Fit、evidence、coverage gap；
- Phase A 候选池生成后必须等待采购员确认初版邀标供方；
- Phase B 固定生成邀标四件套；
- 供方信息长名单只覆盖人工确认初版供方；
- BCC 邮箱必须能回溯到长名单；
- `material-requirement-analysis` 下游改为 `sourcing-invitation`；
- 删除独立 `official-supplier-matching` 入口。

## v0.7.8

- 招标意向征集登记表 N3 投标保证金改为当前项目动态变量；
- 计算：`CEILING(采购清单预估总金额 × 1%, 1000)`；
- 缺少可追溯预估总金额时阻断，不得沿用历史固定金额。

## v0.7.7

- 企业原《招标意向征集登记表.xlsx》成为正式模板；
- 03 交付物必须复制企业原模板并仅填当前项目允许字段；
- 新增 Original Template Integrity Gate。

## v0.7.6

- 邀标邮件升级为单一 `.eml` 草稿；
- 人工确认供方邮箱统一写入 Bcc；
- EML 不嵌入 Excel 附件；
- 后续进一步增加 Markdown-Free Gate。

## v0.7.5

- `material-requirement-analysis` P0 清零后强制生成 Excel；
- 有 P1/P2 输出澄清版，全部确认输出最终版；
- 供应商报价列保持空白。

## v0.7.4

- `historical-procurement-analysis` 不足12个月有效历史数据必须折算为12个月采购基线；
- 无两年度可比数据时由用户确认年度增量路径。

## v0.7.3

- 新增强制 Agreement Selection Gate；
- 用户必须明确选择一个或多个协议编号。

## v0.7.2

- 历史采购仅允许 `已完成 / 执行中` 订单进入统计和预测。

## v0.7.1

- 新增 `.agents/skills/` 作为 Codex 仓库级 Skill 主运行目录；
- `skills/` 保留兼容镜像。

## v0.7.0

- 新增 `sourcing-strategy` 并对接企业《物资采购方案报告模板.docx》。

## v0.6.0

- 新增 `shortlist-approval`。

## v0.5.x

- 新增并持续强化 `historical-procurement-analysis`，包括 SKU+区域、真实订单字段映射与年化规则。

## v0.4.0

- 新增 `supplier-shortlist`。

## v0.3.0

- 新增 `sourcing-invitation` 及邀标交付物。

## v0.2.0

- 新增 `official-supplier-matching`，明确官方供方库 ONLY。

## v0.1.0

- 新增 `material-requirement-analysis`、基础 schemas 和需求诊断案例。
