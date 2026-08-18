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

每次运行先识别：当前项目、已有产物、已完成阶段、blocker、人工 checkpoint、下一阶段。

如果用户提供的是中间阶段产物，应从对应阶段继续，不要求从 Phase 0 重跑。

## 3. Phase State Machine

允许状态：`phase_0_history_optional / phase_1_requirement / phase_2_supplier_sourcing / waiting_supplier_replies / phase_3_shortlist / phase_4_strategy / completed / blocked`。

建议维护 `{{项目名称}}-procurement-workflow-state.yaml`，结构使用 `schemas/property-material-sourcing-workflow.schema.yaml`。

## 4. Phase 0 — Historical Procurement Analysis（Optional）

调用 `historical-procurement-analysis`。有历史采购数据时执行；没有则跳过。协议范围、订单状态、年化、年度增量、人机确认均遵循该模块规则。执行后保留历史测算需求清单及 strategy_report_handoff，并尽可能向 Phase 4 提供整体、区域、业务单元、订单金额段、SKU/Pareto、历史供方、价格结构。

## 5. Phase 1 — Material Requirement Analysis

调用 `material-requirement-analysis`。P0 未解决不得进入 Phase 2；P0 清零后必须生成 Excel；关键字段确认后形成最终需求清单。供应商报价字段保持空白。

## 6. Phase 2 — Official Supplier Sourcing + Invitation

调用 `sourcing-invitation`。

Phase 2A 候选身份只能来自企业内部官方供方库，必须执行品类/区域/资质/状态等 Hard Gate、Fit、Evidence、Coverage Gap。公网资料绝不能新增邀标候选供方。

Human Gate A：AI 候选池 ≠ 正式邀标供方，必须采购员确认初版供方。

Phase 2B 固定生成：
1. `{{项目名称}}-邀标邮件.eml`
2. `{{项目名称}}-最终需求清单.xlsx`
3. `{{项目名称}}-招标意向征集登记表.xlsx`
4. `{{项目名称}}-供方信息长名单.xlsx`

EML 单一 Bcc、不嵌附件、正文无 Markdown；03 使用企业原版模板并按当前规则计算 N3 保证金；04 仅内部使用，不展示供方编码，联系人/电话/邮箱不得猜测。

人工审核/发送后状态进入 `waiting_supplier_replies`。

## 7. Waiting Supplier Replies

不得猜测供方回复。只有真实邮件、回函、登记表或人工记录后才能进入 Phase 3。

## 8. Phase 3 — Supplier Shortlist + Approval

调用 `supplier-shortlist`。

Phase 3A 仅基于实际回复形成建议入围/待澄清/不建议入围和 `{{项目名称}}-供方短名单.xlsx`。

Human Gate C：采购员明确确认最终名单后，Phase 3B 才能固化最终短名单、生成短名单报批邮件和 shortlist-handoff。确认后不得重新评分、排序、增删或替换供方。

## 9. Phase 4 — Sourcing Strategy

调用 `sourcing-strategy`，消费最终需求、Phase 2 可追溯结果、人工确认 shortlist_handoff、历史分析（如有）和当前项目规则。

固定输出：
1. `{{项目名称}}-strategy-data.yaml`
2. `{{项目名称}}-采购方案报告.docx`

必须使用企业采购方案模板。

支出分析按 V3 模板完成整体、军种/业务单元/项目/业态、SKU、免运额度，并补充区域、订单金额段、历史供方、价格等可用维度。

当地供应市场行情至少覆盖内部供应信号、供应格局、成本构成、价格/风险趋势、本地履约因素和采购动作。Web 可用且地区/品类明确时必须主动研究公开市场，但不得改变官方候选池。

不得为了保持原页数压缩分析；空间不足时扩展行高、表格行或自然跨页。

## 10. Global Human Gates

不可自动越过：协议范围选择、必要的年度增量确认、P0 澄清、初版邀标供方确认、邀标人工发送、最终短名单确认、预算/定标/中标家数/份额/目标降本率/评标人员等 Human Decision、最终方案报批。

## 11. Global Guardrails

- V1 只做物业物资类；
- 不从公网新增候选供方；
- 不编造资质、分类、区域覆盖、联系人、邮箱、电话、价格、历史业绩、预算、制度、审批人；
- 不把模板示例或历史案例值当当前项目规则；
- 不混用含税/未税；
- 不把部分周期金额当全年金额；
- AI 推荐不能替代采购员确认；
- 不自动发邮件；
- 不覆盖原始输入；
- 所有重要结论可追溯。

## 12. Artifact Registry / Resume

主 Skill 区分 `source_input / generated_draft / human_confirmed / sent_external / superseded / final`。下游优先使用最新 human_confirmed/final 产物。

用户说“继续”时读取 workflow state 和实际产物，从第一个未完成 Gate/阶段继续。上游事实变化时标记受影响的下游产物为 superseded，并说明需重跑阶段。

## 13. Success Criteria

一个统一入口、五个内部专业模块、Human Gate 保留、企业模板不搬迁、任意 checkpoint 可恢复、市场研究与官方候选池隔离、最终策略满足 V3 内容深度。
