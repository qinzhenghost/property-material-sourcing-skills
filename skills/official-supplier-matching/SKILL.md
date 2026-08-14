---
name: official-supplier-matching
description: 基于已确认的物业物资采购标准需求，仅在企业内部官方供方库中检索、过滤和解释候选供方，形成供采购员确认的邀标沟通候选池。禁止通过公网或第三方数据源新增邀标候选供方。V1适用于物业物资类邀标制采购。
metadata:
  version: "0.2.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  supplier_source: "official-registry-only"
---

# Official Supplier Matching

## 1. Purpose

将 `material-requirement-analysis` 已确认的标准需求，与企业内部官方供方库进行匹配，形成供采购员审核的“邀标沟通候选池”。

本 Skill **不负责**：

- 从互联网发现新供方；
- 替采购员决定邀标名单；
- 判断最终中选供方；
- 用 AI 常识补全官方库没有记录的供应能力。

本 Skill 的核心是：

> Official registry first, evidence before inference, human decision before invitation.

## 2. Parent / Scope

- Domain: 物业物资类采购
- Procurement object: Goods / Materials only
- Sourcing method: 邀标制
- Required upstream: `material-requirement-analysis`
- Default downstream: `sourcing-invitation`
- V1 excludes: 服务类供应商寻源

## 3. Entry Gate

只有同时满足以下条件才允许正式执行供方匹配：

1. `requirement_status = confirmed`
2. 不存在未解决的 P0 需求 blocker
3. 至少存在一个企业官方供方数据源
4. 需求已明确可用于寻源的二级/三级品类，或已有采购员确认的官方分类候选
5. 配送/交付区域已明确

如果缺少其中任一项：

- 不得输出“推荐邀标名单”；
- 只能输出缺口和下一步所需信息。

## 4. Authoritative Data Sources

### Allowed

优先级：

1. 企业正式供应商系统/API/MCP/Connector；
2. 企业正式供应商系统的官方导出；
3. 企业明确指定为官方来源的内部供应商文件。

当前黄金样本：

- `重庆市物资类供应商名录.xlsx / Sheet1`
- `重庆市物资类供应商名录.xlsx / 万科物业临时供应商`
- `物资类供应商三级分类表.xlsx`

### Registry Tier

必须区分数据源层级：

#### `approved_official_registry`

字段完整、可作为正式在库候选来源的官方供方记录。

当前样本中，`Sheet1` 可按此层级进行测试。

#### `temporary_internal_registry`

内部临时供应商记录。

当前样本中，`万科物业临时供应商` 只能按此层级处理。

默认规则：

- 可以被检索和提示；
- 不能仅因存在于临时表就自动通过邀标资格 Gate；
- 必须由企业规则或采购员确认其是否允许进入意向征集/邀标流程。

#### `unknown_internal_registry`

内部来源但权限/资格含义不明确。

不得自动进入候选池，只能提示人工核验。

## 5. Absolute Supplier Source Guardrail

### AI MUST NOT

- 使用 Google/Bing/百度等搜索引擎发现邀标供方；
- 使用企查查/天眼查/行业黄页/电商平台新增候选供方；
- 因为知道某品牌或厂家而自行加入供方；
- 将历史项目中出现但不在当前官方库的公司加入候选；
- 将公网找到的“更优供应商”替换官方库供方；
- 把品牌制造商和在库签约主体混为一谈。

### External information MAY be used only for

在用户明确允许的情况下：

- 商品规格研究；
- 市场行情；
- 价格基准；
- 品类知识；
- 已在官方库中的供方公开信息补充研究。

但即使查到外部信息，也**不能改变候选身份来源**：

> Candidate identity must originate from official registry.

## 6. Source-of-Truth Rules

所有匹配判断必须引用官方字段。

允许使用：

- 供方类别
- 供方分类
- 二级类型
- 三级类型
- 合作业务
- 供方地域级别
- 供方覆盖地域
- 行业资质
- 资质有效期
- 批准日期
- 注册资金
- 官方状态字段（若正式系统提供）
- 历史合作/履约官方字段（若正式系统提供）

禁止从公司名称猜测能力。

例如：

> “重庆东强粮油食品有限公司”

名称看起来像粮油供方，但只有官方字段 `三级类型 = 米面粮油` 才能作为品类匹配证据。

## 7. Required Inputs

### Required requirement inputs

从上游 Handoff 获取：

- project_name
- category_path
- delivery_regions
- required_qualifications（如有）
- supplier_hard_constraints（如有）
- requirement_status

### Required supplier inputs

至少一个官方数据源：

- 正式供方 API / Tool；
- 官方供应商 Excel/CSV；
- 其他经企业确认的官方库导出。

### Optional

- 官方历史采购记录
- 官方履约评价
- 官方黑名单/冻结状态
- 官方品牌授权
- 官方资质附件

没有则标记未知，不能凭空推断。

## 8. Workflow

### Step 1 — Validate Upstream Requirement

检查：

- 是否 confirmed；
- category_path 是否足够明确；
- delivery_regions 是否明确；
- 是否存在供方级硬条件。

如果 category_path 未确认：

- 使用官方三级分类表给出候选映射；
- 等人工确认后再正式匹配。

### Step 2 — Register Official Sources

对每个数据源记录：

- 文件/系统名称；
- sheet / endpoint；
- registry tier；
- 数据日期；
- 可用于何种决策。

如果数据源身份不明确：

`registry_tier = unknown_internal_registry`

并阻止其自动进入候选池。

### Step 3 — Normalize Official Taxonomy

只做用于比对的规范化，不覆盖原始字段。

例如：

- `食品饮品` → `食品饮品`
- `米面粮油、生活小家电` → token: `米面粮油`, `生活小家电`
- `休闲食品,米面粮油` → tokenized list

保留原始文本作为 evidence。

不得因为分隔符不同判成 no_match。

### Step 4 — Normalize Regions

允许进行显式行政区名称规范化：

- 重庆 → 重庆市
- 贵州 → 贵州省

但只做名称归一化。

**严禁能力扩展推断。**

例如：

官方字段：

`供方覆盖地域 = 重庆`

需求：

`重庆市 + 贵州省`

则结果只能是：

`region_match = partial`

不得因为：

- 公司规模大；
- 注册资本高；
- 供方地域级别写“区域”；
- 公司总部在其他省；
- AI 认为物流可以配送；

就自动写成覆盖贵州。

### Step 5 — Material Supplier Gate

V1 只接受：

`供方类别 = 物资类`

如果记录为：

- 服务类
- 非采购类
- 无法判断

则不能自动进入物资候选池。

### Step 6 — Category Gate

优先匹配：

`需求二级分类 ↔ 供方二级类型`

`需求三级分类 ↔ 供方三级类型`

同时允许 `合作业务` 作为补充官方证据。

#### Exact / Compatible

例如需求：

`物资类 → 食品饮品 → 米面粮油`

供方：

`二级类型 = 食品饮品`

`三级类型 = 米面粮油`

→ category = pass

#### Multi-category

供方字段：

`休闲食品,米面粮油`

仍可视为包含 `米面粮油`。

#### Broad-only

如果只匹配到：

`食品饮品`

但没有 `米面粮油`

→ category = partial / needs_human_check

不得自动认定其能供应全部 SKU。

### Step 7 — Region Gate

比较所有需求区域。

#### Full

供方官方覆盖能够明确覆盖全部需求区域。

#### Partial

只能覆盖部分区域。

#### No Match

官方字段明确不覆盖需求区域。

#### Unknown

官方字段缺失或含义无法判断。

当项目需要跨区域统一供货，而没有任何 full match 供方时：

- 不要强行选最高分；
- 输出 `registry_coverage_gap`；
- 要求补充更完整的官方区域供方库，或由采购员决定是否分区域寻源。

### Step 8 — Qualification / Status Gate

只有在以下情况下才能做强制判断：

1. 当前需求明确要求某资质；或
2. 企业官方采购规则明确规定该品类必须具备某资质。

如果只有供方库里存在“行业资质”字段，但没有需求/制度证明该资质是强制门槛：

- 可以作为 evidence；
- 不得自行定义为必须项。

如资质有效期字段明确且已过期：

- 标记 `expired`；
- 不自动宣称供应商违法或不能经营；
- 但必须进入人工核验。

若官方库没有状态字段：

`registry_status = unknown`

不得将“在库”写成“有效”。

### Step 9 — Build Hard Gate Result

每家供方至少生成：

```text
official_source
material_supplier
category
region
qualification
registry_status
```

输出结果：

- `eligible_for_review`
- `partial_match_for_review`
- `needs_human_check`
- `ineligible`

### Step 10 — Fit Analysis

只有通过或部分通过 Hard Gate 的供方才做 Fit 分析。

可以使用官方数据中的：

- 合作业务
- 历史采购
- 履约评价
- 同类项目记录
- 注册资金
- 供方分类
- 区域级别

但 Fit 只是排序辅助，不得覆盖 Hard Gate。

**高 Fit + 区域不满足 ≠ Eligible。**

### Step 11 — Evidence-First Explanation

每一项判断必须给 evidence。

推荐：

```yaml
category:
  result: pass
  evidence:
    - "官方三级类型：米面粮油"

region:
  result: partial
  evidence:
    - "官方覆盖地域：重庆"
    - "需求配送区域：重庆市、贵州省"
```

禁止输出：

> “该供应商全国配送能力较强。”

除非官方数据明确记录。

### Step 12 — Candidate Pool

输出分组，而不是直接输出一个“推荐名单”：

#### A. Full Match

硬条件全部满足，可进入采购员审核。

#### B. Partial Match

存在可解释的部分匹配，比如跨区域覆盖不足。

#### C. Needs Official Data Check

缺状态、资质、区域或临时供应商资格等关键官方信息。

#### D. Excluded

明确不符合物资类别/品类/区域等硬条件。

### Step 13 — Coverage Gap Check

候选池生成后必须反向检查：

- 是否有需求 SKU/品类没有官方供方覆盖；
- 是否有区域没有完整候选；
- 是否只有单一候选导致竞争不足；
- 是否候选都来自同一数据源且数据过旧；
- 是否当前官方库只是某一个城市/区域的局部导出。

发现时必须提示：

`official_registry_coverage_gap`

而不是调用公网补供应商。

### Step 14 — Human Checkpoint

AI输出候选池后必须停在人工确认。

采购员可决定：

- include_for_intention_collection
- exclude
- hold
- request_more_official_data

未确认前：

- 不得进入邀请招标邮件生成；
- 不得把候选称为“已确定邀标供应商”。

### Step 15 — Structured Handoff

采购员确认“邀标沟通对象”后：

```yaml
handoff:
  workflow_stage: supplier_candidates_ready
  supplier_source: official_registry_only
  selected_for_invitation_contact:
    - supplier_id: ...
      supplier_name: ...
  unresolved_blockers: []
  human_decision:
    confirmed: true
  next_skill: sourcing-invitation
```

本项目不设置独立的 `supplier-intention-screening` Skill。人工确认候选供方后，直接进入 `sourcing-invitation`，由该 Skill 一次生成邀标邮件正文、最终需求清单和招标意向征集登记表三个产出物。

## 9. Output Format

### A. 寻源范围摘要

- 需求品类
- 配送区域
- 使用的官方供方库
- 官方库数据层级
- 是否存在官方数据覆盖缺口

### B. Hard Gate 统计

例如：

| 指标 | 数量 |
|---|---:|
| 检索记录 | 157 |
| 品类匹配 | 9 |
| 全区域匹配 | 0 |
| 部分区域匹配 | 9 |
| 需人工核验 | 3 |

### C. 候选供方池

| 供方 | 官方来源 | 品类 | 区域 | 资质/状态 | Fit | 建议动作 |
|---|---|---|---|---|---|---|

### D. 每家供应商证据说明

只引用官方数据与明确的 AI 派生结论。

### E. 官方库覆盖缺口

例如：

- 当前提供的是重庆供方名录；
- 需求同时覆盖贵州；
- 未有足够官方字段证明当前候选覆盖贵州。

### F. Human Decision Required

明确要求采购员确认：

- 哪些进入邀标沟通；
- 是否需要追加其他区域的官方供方库；
- 临时供应商是否允许参与；
- 哪些官方字段需要人工补证。

### G. Structured Handoff

只有人工确认后生成。

## 10. Golden-Case Rules Learned from Current Samples

针对当前“米面粮油”样本必须通过：

1. 企业三级分类表中可确认：`物资类 → 食品饮品 → 米面粮油`。
2. `食用油`、`大米`属于该三级分类下的更细物料参考，不应误当成供方库三级类型字段。
3. 当前 `重庆市物资类供应商名录.xlsx / Sheet1` 中存在多条官方字段包含 `米面粮油` 的记录。
4. 不能仅根据公司名称包含“粮油/食品”判断品类，必须引用 `二级类型/三级类型/合作业务`。
5. 当前需求配送区域为 `重庆市 + 贵州省`。
6. 如果候选官方字段仅为 `供方覆盖地域 = 重庆`，必须标记 `partial_region_match`。
7. 当前样本库没有统一明确的有效/冻结状态字段，不得默认 `registry_status = pass`。
8. 某些记录存在历史资质有效期，但过期字段只能触发人工核验，不能由 AI 扩大成法律结论。
9. `万科物业临时供应商` 不能与字段完整的正式名录记录等价处理。
10. 即使官方库没有跨区域 full match，也不得去公网自动寻找供方。

## 11. Human Review

必须由采购员确认：

- 官方数据源是否允许用于本次邀标；
- 临时供应商能否进入流程；
- 部分区域匹配供方是否进入邀标沟通；
- 是否追加其他区域官方库；
- 候选邀标沟通名单；
- 是否进入下一步邀标沟通。

## 12. Guardrails

- 官方库 ONLY。
- 不生成不存在于官方库的候选供方。
- 不把临时内部供应商自动视为正式邀标供方。
- 不从公司名称推断品类能力。
- 不从“区域级别”推断未记录的覆盖省份。
- 不把缺失官方状态解释为有效。
- 不把资质字段缺失自动解释为不合规，除非需求/制度明确其为门槛。
- 不用 Fit 分数绕过硬条件。
- 不自动决定邀标名单。
- 不自动跳过 Human Checkpoint。

## 13. Review Checklist

- [ ] 上游需求是否 confirmed？
- [ ] 是否只用了内部官方供方数据源？
- [ ] 是否标记每个官方数据源的 registry tier？
- [ ] 是否过滤了非物资类记录？
- [ ] 是否使用官方二级/三级分类字段进行品类匹配？
- [ ] 是否保留多品类字段并正确 token 化？
- [ ] 是否逐区域检查覆盖能力？
- [ ] 是否避免跨区域脑补？
- [ ] 是否区分官方字段与 AI 派生字段？
- [ ] 是否处理缺失/过期资质为人工核验，而不是武断结论？
- [ ] 是否检查官方库本身的覆盖缺口？
- [ ] 是否停在人工确认，而不是自动进入邀标？

## 14. Success Criteria

本 Skill 成功的标准：

1. 候选供方身份 100% 可追溯到内部官方库；
2. 每个品类/区域匹配结论都有官方字段证据；
3. AI 不会因为“看起来合适”而新增或扩展供应商能力；
4. 当官方库覆盖不足时能明确暴露缺口；
5. 采购员可以快速确认下一批邀标沟通对象；
6. 输出可直接交给 `sourcing-invitation`。
