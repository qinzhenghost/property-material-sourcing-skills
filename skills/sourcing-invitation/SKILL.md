---
name: sourcing-invitation
description: 基于已澄清的物业物资采购需求，在企业内部官方供方库中完成供方匹配、Hard Gate、证据说明和候选池分析；经采购员人工确认初版供方后，继续生成单一 BCC 邀标邮件 .eml、最终需求清单.xlsx、企业原版招标意向征集登记表.xlsx 和采购内部供方信息长名单.xlsx。禁止公网新增候选供方，不自动发送邮件。
metadata:
  version: "0.4.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  supplier_source: "official-registry-only"
---

# Sourcing Invitation

## 1. Purpose

本 Skill 合并原 `official-supplier-matching` 与 `sourcing-invitation`，覆盖一个完整业务阶段：

`标准需求 → 官方供方库匹配 → 候选池 → 人工确认初版供方 → 联系信息长名单 → BCC邀标邮件与附件`

分为两个强制阶段：

- **Phase A — Official Supplier Matching**：只使用企业内部官方供方数据，形成供采购员审核的候选池。
- **Phase B — Invitation Package**：只有采购员明确确认初版供方后，才生成正式邀标四件套。

人工确认节点保留在 Skill 内部，不再单独调用 `official-supplier-matching`。

## 2. Scope / Upstream / Downstream

- Domain: 物业物资类采购
- Procurement object: Goods / Materials only
- Sourcing method: 邀标制
- Required upstream: `material-requirement-analysis`
- Default downstream: `supplier-shortlist`
- V1 excludes: 服务类供应商寻源

## 3. Phase A Entry Gate

允许进入官方供方匹配前，至少满足：

1. 上游 `p0_blocker_count = 0`；
2. 已有澄清版或最终需求清单；
3. 至少存在一个企业官方供方数据源；
4. 需求品类已明确到可用于官方分类匹配的层级，或存在采购员待确认的官方分类候选；
5. 配送/交付区域已明确。

如果缺少任一项：

- 不得输出“推荐邀标名单”；
- 只能输出缺口与所需补充信息；
- 不进入 Phase B。

P1/P2 若不影响供方适配与邀标关键条件，可以保留留痕并继续；若会改变品类、区域、资格、商务条件或邮件内容，则必须先确认。

## 4. Authoritative Supplier Sources

候选供方身份只能来自：

1. 企业正式供应商系统 / API / MCP / Connector；
2. 正式供应商系统官方导出；
3. 企业明确指定为官方来源的内部供应商文件。

### Registry Tier

- `approved_official_registry`：正式在库来源，可进入 Hard Gate 与采购员审核。
- `temporary_internal_registry`：内部临时记录，可提示，但不得自动视为具备邀标资格。
- `unknown_internal_registry`：内部来源但权限含义不清，只能人工核验。

### Absolute Guardrail

不得使用以下来源新增邀标候选供方：

- 搜索引擎；
- 企查查 / 天眼查 / 行业黄页；
- 电商平台；
- 品牌官网；
- 历史项目中但不在当前官方库的供应商；
- AI 常识推断出的厂家或经销商。

外部信息最多用于市场研究或已在库供方的公开信息补充，不能改变 `candidate_identity_source`。

详细规则见：

- `references/official-supplier-source-policy.md`
- `references/category-and-region-matching-rules.md`

## 5. Source-of-Truth Rules

供方匹配允许引用的官方字段包括：

- 供方类别 / 供方分类
- 二级类型 / 三级类型
- 合作业务
- 供方地域级别 / 供方覆盖地域
- 行业资质 / 资质有效期
- 批准日期
- 注册资金
- 正式系统状态字段
- 官方历史合作 / 履约字段
- 官方联系人 / 电话 / 邮箱字段

禁止根据公司名称推断业务能力、覆盖区域或资质状态。

## 6. Workflow — Phase A Official Supplier Matching

### Step A1 — Validate Requirement

检查：

- P0 是否为 0；
- category_path 是否足够明确；
- delivery_regions 是否明确；
- 是否存在 required_qualifications / supplier_hard_constraints。

### Step A2 — Register Official Sources

对每个供方数据源记录：

- source_file / system
- source_sheet / endpoint
- registry_tier
- 数据日期（如有）
- 可用于哪些判断

来源权限不明确时：

`registry_tier = unknown_internal_registry`

不得自动进入候选池。

### Step A3 — Normalize Taxonomy

仅做用于匹配的规范化，不覆盖官方原值。

多值字段允许按 `/ , ， 、 ; ；` token 化。

品类匹配优先：

`需求二级分类 ↔ 供方二级类型`

`需求三级分类 ↔ 供方三级类型`

`合作业务` 只能作为补充证据。

匹配等级：

- `exact`
- `compatible`
- `partial`
- `no_match`
- `unknown`

### Step A4 — Normalize Regions

只允许行政区名称的一一规范化，例如：

- 重庆 → 重庆市
- 贵州 → 贵州省

不得将“区域级供方”“注册资本较大”“总部在某地”等扩展推断为未记录的配送能力。

### Step A5 — Material Supplier Gate

V1 只接受：

`供方类别 = 物资类`

服务类、非采购类或无法判断记录不得自动进入物资候选池。

### Step A6 — Category Gate

- 二级 + 三级明确匹配 → pass / exact
- 三级包含需求但表达为多品类 → compatible
- 只匹配二级大类 → partial / needs_human_check
- 明确不一致 → no_match
- 信息不足 → unknown

公司名称不能作为 Gate evidence。

### Step A7 — Region Gate

对全部需求区域逐一检查：

- `full`
- `partial`
- `no_match`
- `unknown`

例如官方字段只有“重庆”，需求为“重庆市 + 贵州省”，只能是 `partial`。

没有任何 full match 时不得强行选最高 Fit，应输出 `official_registry_coverage_gap`。

### Step A8 — Qualification / Status Gate

资质只有在以下情况下作为 Hard Gate：

1. 当前需求明确要求；或
2. 企业正式规则明确要求。

资质过期字段触发人工核验，不扩大成法律结论。

若官方库没有明确有效/冻结/禁用状态：

`registry_status = unknown`

`in registry ≠ active`

### Step A9 — Hard Gate Result

每家供方至少记录：

- official_source
- registry_tier
- material_supplier
- category
- region
- qualification
- registry_status
- evidence

总体结果：

- `eligible_for_review`
- `partial_match_for_review`
- `needs_human_check`
- `ineligible`

### Step A10 — Fit Analysis

只有通过或部分通过 Hard Gate 的供方才进行 Fit 分析。

可以参考官方：

- 合作业务
- 历史采购
- 履约评价
- 同类项目
- 注册资金
- 供方分类
- 区域级别

Fit 只用于辅助阅读/排序，不得绕过 Hard Gate。

### Step A11 — Candidate Pool

候选池必须分组输出：

#### A. Full Match
硬条件完整满足，可进入采购员审核。

#### B. Partial Match
存在明确部分匹配，例如跨区域覆盖不足。

#### C. Needs Official Data Check
状态、资质、区域、临时供方资格等需要补证。

#### D. Excluded
明确不符合物资类别、品类、区域或其他硬条件。

每家供方必须给出官方 evidence。

### Step A12 — Coverage Gap Check

反向检查：

- 是否有品类/SKU 无官方供方覆盖；
- 是否有区域无完整候选；
- 是否只有单一候选导致竞争不足；
- 是否供方库只是局部城市/区域导出；
- 是否关键官方字段普遍缺失或数据过旧。

只能请求更多官方数据，不能转向公网补供方。

### Step A13 — Human Supplier Checkpoint（强制暂停）

AI 输出候选池后必须停下，采购员决定：

- `include_for_intention_collection`
- `exclude`
- `hold`
- `request_more_official_data`

未得到人工确认前：

- 不得生成正式 BCC 邀标邮件；
- 不得生成正式供方信息长名单；
- 不得把候选称为“已确定邀标供应商”；
- 不得进入 Phase B。

人工确认后形成：

`confirmed_initial_supplier_pool`

## 7. Workflow — Phase B Invitation Package

### Step B1 — Validate Initial Supplier Pool

只允许 `confirmed_initial_supplier_pool` 进入邀标准备。

04 长名单和 BCC 都必须严格以该集合为边界，不得新增未确认供方。

### Step B2 — Generate Supplier Contact Longlist（强制）

使用：

`.agents/skills/sourcing-invitation/templates/供方信息长名单模板.xlsx`

生成：

`{{项目名称}}-供方信息长名单.xlsx`

至少包含：

- 序号
- 供方名称
- 供方编码
- 联系人姓名
- 联系电话
- 邮箱地址
- 联系信息来源
- 联系信息状态
- 备注

规则：

1. 覆盖全部人工确认初版供方；
2. 联系人/电话/邮箱仅来自官方库或采购员明确确认信息；
3. 缺失字段保持空白并标记，不得猜测；
4. 同一供方多个已确认联系人时，一条联系人记录一行；
5. 状态可用：`完整 / 缺联系人 / 缺电话 / 缺邮箱 / 多项缺失 / 待人工确认`；
6. 04 是采购内部文件，禁止对外发送或在 EML 正文中引用。

详细规则见：`references/supplier-contact-longlist-rules.md`。

### Step B3 — Build Recipient Set

从 04 长名单读取已确认邮箱：

- 已确认邮箱 → `Bcc`
- 缺失/冲突 → `missing_recipient_emails`
- 供方邮箱不得进入 `To` / `Cc`

每个 BCC 地址必须能回溯到 04 长名单。

冲突时：

`BLOCK/HOLD: supplier_contact_bcc_mismatch`

### Step B4 — Generate Single EML

生成：

`{{项目名称}}-邀标邮件.eml`

规则：

- 每个项目只生成 1 个 EML；
- Bcc = 全部已确认且邮箱已确认的供方；
- To 仅可使用采购方明确提供的本方发送/归档邮箱，否则留空；
- Cc 仅使用采购员明确确认的内部邮箱；
- 正文只引用 02 与 03；
- 不引用 04；
- EML 不嵌入附件；
- 不自动发送。

详细规则见：`references/eml-delivery-rules.md`。

### Step B5 — Prepare Final Requirement Workbook

02 = 人工确认的最终需求清单。

不得重新优化、改数量、改规格、改区域或预填供方报价字段。

### Step B6 — Calculate Bid Bond Variable

读取：

`procurement_estimated_total_amount`

计算：

`raw_bid_bond = procurement_estimated_total_amount × 1%`

`bid_bond_amount = CEILING(raw_bid_bond, 1000)`

即向上取整到 1000 元整数倍。

采购清单预估总金额必须来自可追溯的当前项目数据，禁止从供方待填写报价列推断，也禁止沿用历史固定保证金。

详细规则见：`references/bid-bond-variable-rules.md`。

### Step B7 — Prepare Intention Form from Original Enterprise Template

03 必须复制：

`.agents/skills/sourcing-invitation/templates/招标意向征集登记表模板.xlsx`

流程：

`copy original template → populate allowed project variables → replace N3 variable → save project file`

`Sheet1!N3` 中：

`{{投标保证金金额}} → bid_bond_amount`

必须保持原模板：

- 工作表名称/数量
- 行列结构
- 合并单元格
- 字体/字号/颜色
- 边框/填充/对齐
- 行高/列宽
- 打印设置
- 公式与数据验证
- 供方填写区域
- 企业固定说明与字段结构

禁止重新设计或重建类似表格。

### Step B8 — Integrity & Consistency Check

#### Supplier Matching

- 候选身份 100% 来自官方库；
- 人工确认初版供方是候选池子集；
- 没有公网新增供方；
- 关键匹配判断有官方 evidence。

#### Longlist / BCC

- 04 覆盖全部 confirmed initial suppliers；
- 联系信息来源可追溯；
- 缺失字段已留空标记；
- BCC 与 04 一致。

#### EML

- 供方邮箱仅在 Bcc；
- 不嵌入附件；
- 正文仅引用 02、03；
- 04 不外发。

#### Intention Form

- 从企业原模板副本生成；
- N3 不残留变量；
- N3 = `CEILING(预估总金额 × 1%, 1000)`；
- 供方待填写字段保持空白。

### Step B9 — Output Package

人工确认初版供方后固定输出四个独立文件：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
04 {{项目名称}}-供方信息长名单.xlsx
```

其中：

- 02、03 = 对外发送时由采购员手动添加的附件；
- 04 = 采购内部文件，不外发；
- EML 不内嵌任何附件。

同时输出符合 `schemas/sourcing-invitation-package.schema.yaml` 的 manifest。

### Step B10 — Human Send Checkpoint

生成完成不等于发送。

采购员最终确认：

- 初版供方范围；
- Bcc；
- 邮件正文；
- 02、03 两个实际外发附件；
- 回复截止时间；
- 商务条款；
- 投标保证金；
- 04 长名单联系信息完整性。

只有采购员明确要求发送且运行环境具备邮件工具时，才可执行发送动作。

## 8. Output Contract

### Phase A 尚未人工确认

仅输出：

1. 寻源范围摘要；
2. Hard Gate 统计；
3. 候选供方池；
4. 每家供方 evidence；
5. 官方库覆盖缺口；
6. Human Decision Required。

此时不生成正式邀标四件套。

### Phase B 已人工确认

必须输出：

1. `{{项目名称}}-邀标邮件.eml`
2. `{{项目名称}}-最终需求清单.xlsx`
3. `{{项目名称}}-招标意向征集登记表.xlsx`
4. `{{项目名称}}-供方信息长名单.xlsx`
5. Structured manifest

## 9. Guardrails

- 官方供方库 ONLY。
- 不从公网新增候选供方。
- 不从公司名称推断品类能力。
- 不从区域级别/注册地址推断未记录覆盖区域。
- 不把“在库”自动解释为有效。
- 不用 Fit 绕过 Hard Gate。
- 不把临时内部供方自动视为正式邀标供方。
- 不跳过初版供方人工确认 Gate。
- 不猜联系人、电话或邮箱。
- 供方邮箱只能进入 Bcc。
- 不自动发送邮件。
- EML 不嵌入附件。
- 04 长名单不外发。
- 不改变 confirmed requirement。
- 不替供方填写意愿/能力声明。
- 不重建企业原版招标意向征集登记表。
- 不沿用历史固定保证金。

## 10. Success Criteria

1. 候选供方身份 100% 可追溯到企业内部官方库；
2. 品类/区域/资质等判断有官方字段证据；
3. 官方库覆盖不足时明确暴露缺口而不是公网补供方；
4. AI候选池与人工确认初版供方严格分离；
5. 人工确认后稳定生成四件套；
6. 长名单与 BCC 一致且联系信息不猜测；
7. 03 严格继承企业原模板并正确计算 N3 保证金；
8. 停在人工发送确认节点；
9. 输出可直接交给 `supplier-shortlist` 作为后续供方回复阶段的项目基线。
