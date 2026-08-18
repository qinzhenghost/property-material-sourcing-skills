---
name: sourcing-strategy
description: 将已确认的物业物资采购需求、历史采购分析、官方供方资源、人工确认的供方短名单及项目采购规则整合为采购方案/采购策略报告。使用企业现有《物资采购方案报告模板》，所有结论必须可追溯到内部数据、采购员确认、前序 Skill 计算结果或有明确来源的公开市场信息。
metadata:
  version: "0.7.2"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  stage: "procurement-strategy"
---

# Sourcing Strategy

## 1. Purpose

本 Skill 用于生成物业物资采购项目的《采购方案报告》。

它不是从空白开始写报告，而是消费前序 Skill 的结构化结果：

```text
historical-procurement-analysis（可选）
        +
material-requirement-analysis
        +
sourcing-invitation
        +
supplier-shortlist（已完成人工确认的 strategy_handoff）
        +
当前项目采购规则 / 人工确认
        ↓
sourcing-strategy
        ↓
采购方案报告.docx
```

核心原则：

> Assemble verified procurement facts into the enterprise template, but do not reduce analysis sections to headings or placeholders.

## 2. Entry Gate

生成“可报批版”前，至少应满足：

1. 需求状态已确认；
2. 项目名称、采购周期/需求时间明确；
3. 采购方式明确；
4. 预算金额或预算形成逻辑明确；
5. 配送范围明确；
6. `supplier-shortlist` 的最终短名单已由采购员确认；
7. 供应商选择方法/定标规则已确认；
8. 关键商务条款已确认。

如部分字段缺失，可以生成 `draft`，但不得输出为 `ready_for_approval`。

## 3. Inputs

### Required

- `material-requirement-analysis` 的 confirmed requirement；
- `supplier-shortlist.strategy_handoff`，且 `human_confirmation.shortlist_confirmed = true`；
- 当前企业采购方案模板；
- 当前项目采购方式及关键采购规则。

### Optional

- `historical-procurement-analysis.strategy_report_handoff`；
- `historical-procurement-analysis` 的结构化历史记录、projection、价格分析结果；
- `sourcing-invitation` 的官方供方匹配 / coverage gap / bid_bond / supplier response 结果；
- 项目预算；
- 历史价格；
- 当前市场价格/成本信息；
- 行业政策/国标/监管要求；
- 项目风险补充；
- 评标小组成员；
- 招标计划节点。

## 4. Source Hierarchy

### Tier A — Internal confirmed facts

优先级最高：

- 已确认需求清单；
- 历史订单/采购数据；
- 官方供方库；
- 已确认短名单；
- 企业采购规则；
- 采购员明确确认的信息。

### Tier B — Calculated facts

例如：

- 历史采购金额；
- SKU支出占比；
- 区域支出占比；
- 供应商支出占比；
- 同比增长率；
- 月均支出；
- 建议采购量；
- 建议采购金额；
- 供方数量统计；
- `sourcing-invitation` 已按当前项目规则计算的投标保证金。

必须保留计算依据。不得在本 Skill 内重新发明与前序结果不一致的统计口径。

### Tier C — Public market intelligence

仅允许用于：

- 当地供应市场行情；
- 原材料/成本驱动；
- 行业供需趋势；
- 公开标准/政策；
- 公开价格趋势；
- 市场风险。

**不得使用公开网络新增邀标候选供方。**

公开市场信息必须保留：source、source_date、evidence_summary、confidence。

如果执行环境具备 Web / Internet 检索能力，且项目品类与配送地区已明确，则“当地供应市场行情分析”必须主动检索公开信息，不能仅凭常识写泛化描述。

公开信息优先级：

1. 政府部门、统计局、发改/价格监测、监管部门、国家/地方标准；
2. 行业协会、交易所/公开价格指数、头部生产企业公开信息；
3. 可信行业研究、主流 B2B / 电商公开价格信息（只作为市场价格与供给佐证，不进入邀标池）。

### Tier D — Human decision

以下内容不能由 AI 擅自决定：

- 最终预算；
- 中标供方数量；
- 定标方法；
- 份额划分；
- 目标降本率；
- 评标小组成员；
- 最终风险处置方案；
- 当前项目保证金规则的变更。

如果前序 `sourcing-invitation` 已按采购员确认规则计算保证金，策略报告应消费该 Calculated Fact，不得再次自行改值。

## 5. Template Mapping Workflow

使用 `references/procurement-plan-template-mapping.md`。

### Step 1 — 采购项目名称
来源：confirmed requirement.project_name。不得从历史案例复制项目名称。

### Step 2 — 采购方式
V1 项目默认业务模式为邀标，但报告中的勾选/表述仍应以当前项目确认值为准。

### Step 3 — 需求时间
来源：当前采购计划、用户明确提供的目标时间。缺失时标记 `MISSING`。

### Step 4 — 预算金额
优先来源：
1. 采购员确认预算；
2. 历史采购分析形成的预测金额；
3. 已确认需求量 × 已确认预算/参考单价。

不得将历史部分周期采购额直接当年度预算，不得混用含税/未税口径。

### Step 5 — 需求背景简介
整合采购使用场景、覆盖区域、原协议情况、历史采购趋势、需求增长依据、本次续采/集采背景。必须以真实数据为主。

### Step 6 — 标段或标的划分
来源：需求清单、区域划分、SKU结构、采购员确认的标段策略。不得从历史模板复制标段数量。

### Step 7 — 支出分析（强制深度分析）

`支出分析` 是采购方案的核心分析章节，不允许只输出“整体 / 区域 / SKU / 价格”标题，也不允许因为部分维度缺失就整段省略。

#### 7.1 有历史采购数据时

必须优先消费 `historical-procurement-analysis.strategy_report_handoff` 及其可追溯结构化结果，并尽可能完成以下分析：

1. **整体支出**
   - 历史数据覆盖期间；
   - 有效订单/记录口径；
   - 历史实际采购金额；
   - 如为部分周期，必须同时给出实际金额和前序 Skill 已确认的年化基线/预测金额；
   - 月均支出或周期平均支出（仅在前序数据支持时）；
   - 同比/环比/增长率（仅在可比口径存在时）；
   - 含税/未税口径。

2. **区域 / 项目 / 业态维度**
   - 数据中存在该维度时，至少列出主要维度的采购金额、占比和差异；
   - 至少展示 Top 3，若不足 3 个则全部展示；
   - 识别高集中区域、异常增长区域或履约压力区域；
   - 模板中的“军种维度”只有在数据确实存在对应业务维度时才填写，不能留下空标题。

3. **SKU 维度**
   - 至少分析 Top 5 高支出 SKU（不足 5 个则全部）；
   - 应包含可获得的采购金额、支出占比、采购量、历史平均单价；
   - 给出 Top 5 / Top 10 支出集中度（前序结果可计算时）；
   - 识别关键 SKU、长尾 SKU、数量高但金额低、金额高但数量低等结构性特征。

4. **供应商维度**
   - 历史数据存在供应商字段时，分析主要历史供方支出及集中度；
   - 至少列示 Top 3 历史供方的采购金额/占比（不足 3 家则全部）；
   - 判断是否存在单一供方或高集中度风险；
   - 该分析只描述历史支出，不得据此自动加入当前邀标短名单。

5. **价格维度**
   - 对可比 SKU 分析历史平均价、价格变化、异常高低价或波动；
   - 可使用加权平均价、最低/最高价、中位价等，但必须确保来自前序计算或当前可追溯数据；
   - 不得把不同规格、品牌、税率、单位的价格直接混比。

6. **结论与采购含义**
   - 至少形成 3 条有数字证据的关键发现；
   - 至少形成 2 条采购策略含义，例如重点议价 SKU、区域履约策略、价格锁定、备选供方、批量集采机会等；
   - 每条结论必须能回指具体金额、占比、数量、价格或趋势证据。

#### 7.2 无完整历史数据时

不得简单写“暂无数据”后结束。应按以下优先级降级：

1. 有当前需求量 + 预算/参考价：生成**计划支出结构**，分析 SKU/区域预计金额与占比；
2. 有当前需求量但无参考价：分析数量结构、关键 SKU 集中度，并明确“金额支出分析暂不可计算”；
3. 有历史文件但没有 `historical-procurement-analysis` handoff：不得静默跳过，应标记需要先执行/补充历史采购分析；
4. 完全无支出依据：列明缺失字段、影响和补充建议，`spend_analysis.completeness_status = insufficient_data`。

#### 7.3 支出分析禁止项

- 不得只复制模板标题；
- 不得保留空表、`X`、`XX`、`X%` 等占位符；
- 不得为了“看起来完整”虚构金额、占比、增长率；
- 不得把部分周期金额直接当全年金额；
- 不得在税口径不一致时直接比较。

### Step 8 — 当地供应市场行情分析（强制研究）

该章节必须同时回答“本地有没有足够供应、成本为什么变化、价格可能往哪里走、对本次采购有什么影响”，不能只写泛化市场描述。

#### 8.1 内部供应市场信号

若前序数据存在，必须优先纳入：

- 官方供方库中匹配当前品类的供方数量；
- 明确覆盖当前配送区域的供方数量；
- 实际邀约/沟通数量、明确回复数量、短名单数量；
- 对账期、配送范围、全量 SKU、保证金等关键商务条件的接受情况；
- coverage gap / 区域履约缺口。

这些信息用于描述**当前企业可触达供应市场的充分程度**。

#### 8.2 外部公开市场研究

当 Web 可用时，至少覆盖以下 5 个主题：

1. **供应格局**：本地市场以生产商、经销商、综合贸易商、电商渠道还是区域配送商为主，供应是否分散；
2. **上游与成本驱动**：识别与当前品类真正相关的原材料、制造、包装、人工、仓储、物流等因素；
3. **价格趋势**：说明近期主要原料/成品价格走势，能量化则量化，不能量化则给方向与证据；
4. **本地履约因素**：配送半径、仓储、零散订单、搬运、项目分散度、时效等对报价的影响；
5. **政策/标准因素**：与本品类直接相关的国家/地方标准、监管要求、环保/安全/质量要求。

公开市场研究原则：

- 原则上至少使用 3 个相互独立的公开来源；
- 优先使用近 12 个月资料；若只能使用更早资料，必须标注日期与时效风险；
- 每个重要结论保留 `source / source_date / evidence_summary / confidence`；
- 公开市场中发现的供应商只能作为“市场结构证据”，不得进入邀标候选池。

#### 8.3 成本构成

- 只有存在可信公开证据时才能写具体成本百分比；
- 若无法取得可靠比例，应写成定性结构，例如“材料成本为主要驱动、物流和人工对零散配送报价形成附加影响”；
- **严禁保留模板中的 `原物料X% / 人工X% / 利润X% / 税率X%` 作为正式分析内容。**

#### 8.4 采购策略影响

至少输出 3 条与本项目直接相关的采购含义，例如：

- 是否适合锁价/设置调价机制；
- 哪些高支出 SKU 应作为重点议价对象；
- 是否需要备选供方或跨区域履约验证；
- 是否需要设置电商/公开指数价格校验机制；
- 是否应拆分标段、合并订单、设置最低配送金额；
- 是否存在原材料价格波动导致的合同风险。

不能只写“市场竞争充分，建议择优采购”这类无证据泛化结论。

#### 8.5 无 Web / 无可靠市场资料时

仍须输出内部供应市场信号 + 已知成本驱动 + 数据缺口；同时标记 `market_analysis.completeness_status = public_research_incomplete`。不得虚构公开行情。

### Step 9 — 供应商短名单及入围理由

只能消费：

- `supplier-shortlist.strategy_handoff` 中 `human_confirmation.shortlist_confirmed = true` 的最终结果。

推荐输出：

```text
目前官方供方库内本品类共识别X家候选供方，
本轮实际沟通X家，
明确回复X家，
经采购员确认后最终短名单X家。
```

再简述可追溯入围理由。

不得：

- 添加未进入官方候选池的公网供应商；
- 使用 Phase A 的 AI 建议替代人工最终结果；
- 修改采购员已确认短名单；
- 虚构供方资质/业绩。

### Step 10 — 交货期
来源：confirmed requirement、邀标条件、人工确认。不得直接复制历史案例。

### Step 11 — 质保期
只有当前品类确实涉及质保且有明确要求时填写；无统一要求时写“按需求清单/合同约定”。

### Step 12 — 保证金

优先级：

1. `sourcing-invitation.bid_bond` 当前项目计算结果；
2. 当前企业规则 / 采购员明确确认；
3. 缺失则标记 MISSING。

当前默认项目规则如已在前序确认：

`bid_bond = CEILING(procurement_estimated_total_amount × 1%, 1000)`

策略报告只能引用前序可追溯计算结果，不重新计算出不同数值，也不得从历史模板沿用固定金额。

### Step 13 — 供应商选择方法
来源：当前采购方式、定标规则、是否技术标/商务标、中标供方数量、备选供方机制。

### Step 14 — 目标价格
优先依据历史成交价、预算价、市场参考价、开标后对标机制、采购员确认的降本目标。模板示例降本率不得自动继承。

### Step 15 — 定标规则
AI可以整理已确认规则和检查冲突，但不能自行决定中标家数、淘汰阈值或最低价必然中标。

### Step 16 — 份额划分
只有中标供方 ≥2 家时需要；多家时必须由采购员提供/确认分配逻辑。

### Step 17 — 风险防范
至少检查需求预测、价格波动、单一供应商、区域履约、关键SKU、质量/资质、账期接受、数据不完整等风险，并输出证据、影响、建议措施、是否需人工确认。

### Step 18 — 评标小组成员
只能来自当前项目正式人员安排，不得复制历史报告人员。

## 6. Analysis Completeness Gate

在生成 DOCX 前必须先检查：

### Spend Analysis Gate

满足以下任一状态：

- `complete`：历史/计划支出数据足够，整体 + 至少两个可用维度 + 关键结论 + 采购含义已完成；
- `partial`：只有部分维度可分析，但已明确缺失原因并完成所有可用分析；
- `insufficient_data`：缺少金额/价格/历史结果，无法形成有效支出分析。

若存在历史采购 handoff，却没有进入报告支出分析，则直接失败：

`BLOCK: historical procurement evidence not consumed by spend analysis`

### Market Analysis Gate

满足以下任一状态：

- `complete`：内部供应信号 + 外部公开市场研究 + 采购含义均完成；
- `partial`：其中一类信息不足，但已完成可获得部分并明确缺口；
- `public_research_incomplete`：执行环境无 Web 或缺少可靠公开来源。

当 Web 可用、地区与品类明确，却没有任何公开来源时，不得标记 `complete`。

### Placeholder Gate

最终 DOCX 不得残留：

- `X` / `XX` / `XXX`；
- `X%`；
- “待确认”以外的模板示例值；
- 空的“整体 / SKU维度 / 市场情况 / 成本构成情况”等分析标题；
- 被错误拼接到章节标题中的供应商名称或其他字段内容。

发现模板字段错位时必须修复映射，不得以“内容已写入”视为成功。

## 7. Report Field State

每个主要字段内部标记：`FACT / CALCULATED / PUBLIC_MARKET_INTELLIGENCE / HUMAN_DECISION / MISSING`。

## 8. Consistency Gates

### Quantity consistency
报告预计采购量 = 已确认需求清单数量。

### Budget consistency
报告预算必须与当前预算或当前需求量×参考价测算一致。

### Region consistency
报告配送范围 = 最终需求清单/邀标条件。

### Shortlist consistency
报告短名单 = `supplier-shortlist.strategy_handoff` 的人工确认名单。

### Bid bond consistency
报告保证金 = `sourcing-invitation.bid_bond.calculated_amount`（如该字段已形成且规则未被人工变更）。

### Commercial terms consistency
交期、账期、税率、保证金、合作期限必须与当前项目条件一致。

### Award rule consistency
供应商选择方法、定标规则、中标家数、份额划分之间不能互相矛盾。

## 9. Output

### A. Procurement strategy data
`{{项目名称}}-strategy-data.yaml`

包含全部报告字段、数据来源、evidence、confidence、missing items、human decision items，并包含：

- `spend_analysis.completeness_status`
- `spend_analysis.key_findings`
- `spend_analysis.sourcing_implications`
- `market_analysis.completeness_status`
- `market_analysis.internal_supply_signals`
- `market_analysis.sources`
- `market_analysis.sourcing_implications`

### B. Procurement plan report
`{{项目名称}}-采购方案报告.docx`

必须基于 `templates/物资采购方案报告模板.docx`，尽量保留企业原版式，报批前人工审核。

## 10. Draft Status

### `draft`
存在 MISSING、未确认预算、未确认短名单、未确认定标规则、`spend_analysis = insufficient_data`、关键市场研究明显不完整或其他关键字段缺失。

### `ready_for_approval`
只有关键字段有事实/明确人工决策、一致性检查通过、支出分析与市场分析达到可用深度、无P0阻断项时才能标记。

## 11. Guardrails

- 不复用历史案例具体项目数据；
- 不把模板示例值当企业当前规则；
- 不虚构市场成本比例；
- 不从公网新增候选供方；
- 不把市场公开供应商等同于邀标候选供方；
- 不覆盖采购员确认的需求量或短名单；
- 不混用含税/未税金额；
- 不把部分周期历史数据当年度数据；
- 不自行确定预算、保证金规则变更、降本率、定标规则、份额或评标人员；
- 不用空标题、占位符或泛化套话冒充分析；
- 所有重要结论必须可追溯。

## 12. Success Criteria

1. 报告结构与企业模板一致；
2. 报告主要数据可追溯到前序 Skill 或明确来源；
3. 历史采购分析进入支出分析和需求背景；
4. 支出分析至少形成可验证的整体结论、维度分析、关键发现与采购含义；
5. 当地供应市场行情至少形成供应格局、成本驱动、价格/履约因素和采购影响；
6. 短名单与 `supplier-shortlist` 人工确认结果完全一致；
7. 市场信息不会污染官方供方候选池；
8. 数量、预算、区域、保证金和商务条件内部一致；
9. 最终 DOCX 无模板占位符、空分析标题和字段错位；
10. 采购员只需要做关键决策确认，而不是重新整理整份报告。
