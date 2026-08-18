---
name: property-material-sourcing
description: 物业物资类邀请招标采购的一体化主 Skill。统一编排历史采购分析、需求分析、官方供方匹配与邀标、供方回复与短名单、采购策略报告五个专业模块；保留所有 Human Gate、企业模板、官方供方库限制和可追溯证据链。
metadata:
  version: "0.1.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  stage: "end-to-end-orchestration"
---

# Property Material Sourcing

## 1. Purpose

这是仓库的唯一推荐运行入口。

用户不需要再手动判断应该调用 5 个 Skill 中的哪一个。主 Skill 根据当前输入、已有产物和人工确认状态自动判断当前阶段，并调用对应内部模块。

统一流程：

```text
项目输入
  ↓
Phase 0 历史采购分析（有历史数据时）
  ↓
Phase 1 需求分析与最终需求确认
  ↓
Phase 2 官方供方匹配 + 人工确认 + 邀标准备
  ↓
【等待供方实际回复】
  ↓
Phase 3 供方回复分析 + 人工确认最终短名单 + 报批
  ↓
Phase 4 采购策略/采购方案报告
  ↓
完成
```

内部模块：

1. `.agents/skills/historical-procurement-analysis/SKILL.md`
2. `.agents/skills/material-requirement-analysis/SKILL.md`
3. `.agents/skills/sourcing-invitation/SKILL.md`
4. `.agents/skills/supplier-shortlist/SKILL.md`
5. `.agents/skills/sourcing-strategy/SKILL.md`

这些模块仍是各业务阶段的权威规则来源；本主 Skill 负责统一入口、状态判断、阶段衔接、Human Gate 和产物一致性。

## 2. Core Orchestration Principle

> Continue from the latest verified checkpoint; never restart or silently bypass a human decision.

每次运行先识别：

- 当前项目名称；
- 已存在的本项目产物；
- 已完成的阶段；
- 尚未解决的 blocker；
- 已完成人工确认的 checkpoint；
- 当前可继续执行的下一阶段。

如果用户提供的是某个中间阶段产物，例如最终需求清单、供方回复或已经确认的短名单，应从对应阶段继续，不要求从 Phase 0 重跑。

## 3. Phase State Machine

允许状态：

- `phase_0_history_optional`
- `phase_1_requirement`
- `phase_2_supplier_sourcing`
- `waiting_supplier_replies`
- `phase_3_shortlist`
- `phase_4_strategy`
- `completed`
- `blocked`

每次运行建议维护：

`{{项目名称}}-procurement-workflow-state.yaml`

结构使用 `schemas/property-material-sourcing-workflow.schema.yaml`。

状态文件只记录流程状态和产物引用，不替代各阶段正式 Handoff。

## 4. Phase 0 — Historical Procurement Analysis（Optional）

调用：`historical-procurement-analysis`

### When to run

存在可追溯历史订单/采购数据，并且用户希望基于历史数据测算需求或形成支出分析时执行。

### When to skip

没有历史采购数据时直接进入 Phase 1，不得制造历史数据。

### Mandatory gates

- 协议编号必须由用户明确选择；
- 仅 `已完成 / 执行中` 进入统计；
- 不足 12 个月按模块规则折算 12 个月基线；
- 缺少两年度可比数据时，增量率必须按模块规则由用户确认或补充依据。

### Required outputs when executed

- `{{项目名称}}-历史测算需求清单.xlsx`
- historical handoff / `strategy_report_handoff`

Phase 0 的多维历史分析必须尽可能为 Phase 4 提供：整体、区域、业务单元/军种/业态/项目、订单金额段、SKU/Pareto、历史供方集中度和价格结构。

## 5. Phase 1 — Material Requirement Analysis

调用：`material-requirement-analysis`

目标：把原始需求或 Phase 0 的测算结果转为可用于寻源的最终需求。

### Gate

- P0 未解决：只能诊断和澄清，不得进入 Phase 2；
- P0 清零后必须生成 Excel；
- 有 P1/P2：生成澄清版并保留问题；
- 关键字段确认后：生成最终需求清单。

### Output

- `{{项目名称}}-澄清版需求清单.xlsx` 或
- `{{项目名称}}-最终需求清单.xlsx`

供应商报价字段必须保持空白，除非进入后续供方实际回复处理。

## 6. Phase 2 — Official Supplier Sourcing + Invitation

调用：`sourcing-invitation`

### Phase 2A — Official supplier matching

候选供方身份只能来自企业内部官方供方库。

必须执行模块中的：

- Registry Tier；
- 物资类 Gate；
- 品类 Gate；
- 区域 Gate；
- 资质/状态 Gate；
- Fit；
- Evidence；
- Coverage Gap。

公网资料绝不能新增邀标候选供方。

### Human Gate A — Initial supplier pool

AI 候选池 ≠ 正式邀标供方。

必须暂停等待采购员确认初版邀标供方。

未确认时：

`BLOCK: initial_supplier_pool_not_human_confirmed`

### Phase 2B — Invitation package

确认初版供方后固定生成：

1. `{{项目名称}}-邀标邮件.eml`
2. `{{项目名称}}-最终需求清单.xlsx`
3. `{{项目名称}}-招标意向征集登记表.xlsx`
4. `{{项目名称}}-供方信息长名单.xlsx`

规则：

- 单一 `.eml`；
- 供方邮箱只进入 Bcc；
- EML 不嵌入 Excel 附件；
- EML 正文不得残留 Markdown 标记；
- 03 必须基于企业原版 Excel 模板生成；
- N3 投标保证金按当前项目可追溯预估总金额 ×1%，并向上取整至 1000 元整数倍；
- 04 仅为内部文件，展示供方名称、联系人、电话、邮箱、来源、状态、备注，不展示供方编码；
- 缺失联系人/电话/邮箱不得猜测。

### Human Gate B — Send

Skill 只生成草稿和附件，不自动发送。人工审核/发送后进入：

`waiting_supplier_replies`

## 7. Waiting Supplier Replies

这是外部等待状态，不允许用 AI 猜测供方回复。

只有用户提供真实邮件、回函、意向登记表或明确人工记录后，才能进入 Phase 3。

## 8. Phase 3 — Supplier Shortlist + Approval

调用：`supplier-shortlist`

### Phase 3A — Reply analysis

仅基于实际回复：

- 回复/未回复；
- 参与意愿；
- SKU/需求覆盖；
- 账期；
- 配送；
- 保证金；
- 资料完整度；
- 其他已验证商务条件。

AI 只可给：

- 建议入围；
- 待澄清；
- 不建议入围。

生成：`{{项目名称}}-供方短名单.xlsx`

### Human Gate C — Final shortlist

AI 建议 ≠ 最终短名单。

必须由采购员明确确认最终入围/不入围/待澄清结果。

### Phase 3B — Approval package

人工确认后：

1. 固化最终 `{{项目名称}}-供方短名单.xlsx`；
2. 生成 `{{项目名称}}-短名单报批邮件.md`；
3. 生成 `{{项目名称}}-shortlist-handoff.yaml`。

确认后禁止重新评分、重新排序、增加或替换供方。

## 9. Phase 4 — Sourcing Strategy

调用：`sourcing-strategy`

输入至少包括：

- 最终需求；
- Phase 2 官方供方/保证金等可追溯结果；
- Phase 3 人工确认的最终 shortlist_handoff；
- Phase 0 历史分析（如执行）；
- 当前项目采购规则/人工确认项。

固定输出：

1. `{{项目名称}}-strategy-data.yaml`
2. `{{项目名称}}-采购方案报告.docx`

必须使用企业 `物资采购方案报告模板.docx`。

### Spend Content Gate

支出分析必须按 V3 模板完成：

1. 整体；
2. 军种/业务单元/项目/业态（按真实字段映射）；
3. SKU；
4. 免运额度/最低配送金额。

并优先补充区域、订单金额段、历史供方、价格等可用维度。

有足够历史数据时，至少满足 sourcing-strategy v0.7.4 的数字事实、表格、关键发现和采购动作最低深度；数据不足时按降级规则处理，不能只写“暂无数据”。

### Market Content Gate

当地供应市场行情必须至少覆盖：

- 内部供方市场信号；
- 市场情况/供应格局；
- 成本构成；
- 价格或风险趋势；
- 本地配送/仓储/履约因素；
- 对本次采购的具体动作。

Web 可用且地区/品类明确时必须主动研究公开市场；公开市场信息只用于行情研究，不能改变官方候选池。

### Template Layout Rule

不得为了保持原模板页数而删减核心分析。空间不足时，应在保持企业原版式风格的前提下增加行高、复制同样式表格行或自然跨页。

## 10. Global Human Gates

以下 Gate 不能由主 Skill 自动越过：

1. 历史协议范围选择；
2. 需要时的年度增量确认；
3. P0 需求澄清；
4. 初版邀标供方确认；
5. 邀标邮件/附件人工发送；
6. 最终短名单确认；
7. 最终预算、定标方法、中标家数、份额、目标降本率、评标人员等 Human Decision；
8. 最终采购方案报批。

## 11. Global Source Hierarchy

优先级：

1. 当前项目人工确认事实；
2. 当前企业正式模板/规则；
3. 当前项目内部数据及前序 Handoff；
4. 可追溯计算结果；
5. 有明确来源的公开市场情报；
6. 模型一般知识只能用于解释，不能替代事实字段或企业规则。

## 12. Global Guardrails

- V1 只处理物业物资类采购；
- 不从公网新增候选供方；
- 不编造资质、分类、区域覆盖、联系人、邮箱、电话、价格、历史业绩、预算、制度、审批人；
- 不把模板示例值当当前项目规则；
- 不把历史案例具体数据带入新项目；
- 不混用含税/未税；
- 不把部分周期金额当全年金额；
- 不将 AI 推荐替代采购员确认；
- 不自动发邮件；
- 不覆盖原始输入文件，输出必须另存；
- 所有重要结论必须可追溯。

## 13. Artifact Registry

主 Skill 维护当前项目产物清单，并区分：

- `source_input`
- `generated_draft`
- `human_confirmed`
- `sent_external`
- `superseded`
- `final`

下游永远优先消费最新的 `human_confirmed/final` 产物，不得引用已被确认版本替代的旧草稿。

## 14. Resume Rules

用户说“继续”时：

1. 读取最新 workflow state；
2. 检查实际已有产物和人工确认；
3. 找到第一个未完成 Gate/阶段；
4. 从那里继续；
5. 不重新执行已经人工确认且输入未变化的阶段。

如果上游事实发生变化，必须标记受影响的下游产物为 `superseded`，并说明需要重跑哪些阶段。

## 15. Success Criteria

1. 用户只需要调用一个主 Skill；
2. 5 个内部模块按同一项目状态连续衔接；
3. Human Gate 不因整合而消失；
4. 所有企业模板继续使用原模块目录中的已验证文件；
5. 中间产物可被后续阶段自动识别和复用；
6. 公网市场研究与官方供方候选池完全隔离；
7. 最终采购策略达到 V3 模板内容深度；
8. 项目可以在任一 checkpoint 安全暂停和恢复。
