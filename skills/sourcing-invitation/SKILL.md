---
name: sourcing-invitation
description: 基于已澄清的物业物资采购需求，在企业内部官方供方库中完成供方匹配、Hard Gate、证据说明和候选池分析；经采购员人工确认初版供方后，继续生成单一 BCC 邀标邮件 .eml、最终需求清单.xlsx、企业原版招标意向征集登记表.xlsx 和采购内部供方信息长名单.xlsx。最终 EML 正文必须为纯文本或标准 HTML，不得保留 Markdown 格式标记。禁止公网新增候选供方，不自动发送邮件。
metadata:
  version: "0.4.2"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  supplier_source: "official-registry-only"
---

# Sourcing Invitation

## 1. Purpose

本 Skill 覆盖完整的官方供方寻源与邀标准备阶段：

`标准需求 → 官方供方库匹配 → 候选池 → 人工确认初版供方 → 联系信息长名单 → BCC邀标邮件与附件`

分为两个强制阶段：

- **Phase A — Official Supplier Matching**：只使用企业内部官方供方数据，形成供采购员审核的候选池。
- **Phase B — Invitation Package**：只有采购员明确确认初版供方后，才生成正式邀标四件套。

人工确认节点必须保留在 Skill 内部。

## 2. Scope / Upstream / Downstream

- Domain: 物业物资类采购
- Procurement object: Goods / Materials only
- Sourcing method: 邀标制
- Required upstream: `material-requirement-analysis`
- Default downstream: `supplier-shortlist`
- V1 excludes: 服务类供应商寻源

## 3. Phase A Entry Gate

进入官方供方匹配前至少满足：

1. `p0_blocker_count = 0`；
2. 已有澄清版或最终需求清单；
3. 至少存在一个企业官方供方数据源；
4. 需求品类已明确到可用于官方分类匹配的层级，或存在采购员待确认的官方分类候选；
5. 配送/交付区域已明确。

缺少任一项时：

- 不得输出“推荐邀标名单”；
- 只能输出缺口与所需补充信息；
- 不进入 Phase B。

## 4. Authoritative Supplier Sources

候选供方身份只能来自：

1. 企业正式供应商系统 / API / MCP / Connector；
2. 正式供应商系统官方导出；
3. 企业明确指定为官方来源的内部供应商文件。

Registry Tier：

- `approved_official_registry`
- `temporary_internal_registry`
- `unknown_internal_registry`

### Absolute Guardrail

不得使用以下来源新增邀标候选供方：

- 搜索引擎；
- 企查查 / 天眼查 / 行业黄页；
- 电商平台；
- 品牌官网；
- 历史项目中但不在当前官方库的供应商；
- AI 常识推断出的厂家或经销商。

外部公开信息最多用于市场研究或已在库供方的信息补充，不能改变 `candidate_identity_source`。

详细规则见：

- `references/official-supplier-source-policy.md`
- `references/category-and-region-matching-rules.md`

## 5. Source-of-Truth Rules

供方匹配允许引用的官方字段包括：

- 供方类别 / 供方分类
- 二级类型 / 三级类型
- 合作业务
- 供方地域级别 / 覆盖地域
- 行业资质 / 资质有效期
- 批准日期
- 注册资金
- 正式系统状态字段
- 官方历史合作 / 履约字段
- 官方联系人 / 电话 / 邮箱字段

禁止根据公司名称推断业务能力、覆盖区域、资质状态或联系信息。

## 6. Workflow — Phase A Official Supplier Matching

### Step A1 — Validate Requirement

检查 P0、category_path、delivery_regions、required_qualifications、supplier_hard_constraints。

### Step A2 — Register Official Sources

记录 source_file/system、source_sheet/endpoint、registry_tier、数据日期及可用于哪些判断。

### Step A3 — Normalize Taxonomy

只做用于匹配的规范化，不覆盖官方原值。

品类匹配优先：

- 需求二级分类 ↔ 供方二级类型
- 需求三级分类 ↔ 供方三级类型
- 合作业务仅作补充证据

匹配等级：`exact / compatible / partial / no_match / unknown`。

### Step A4 — Normalize Regions

只允许行政区名称的一一规范化，例如“重庆 → 重庆市”“贵州 → 贵州省”。
不得扩展推断未记录的配送能力。

### Step A5 — Material Supplier Gate

V1 只接受物资类供方。服务类、非采购类或无法判断记录不得自动进入候选池。

### Step A6 — Category Gate

- 二级 + 三级明确匹配 → pass / exact
- 三级包含需求但表达为多品类 → compatible
- 只匹配二级大类 → partial / needs_human_check
- 明确不一致 → no_match
- 信息不足 → unknown

公司名称不能作为 Gate evidence。

### Step A7 — Region Gate

对全部需求区域逐一检查：`full / partial / no_match / unknown`。

例如官方字段只有“重庆”，需求为“重庆市 + 贵州省”，只能是 `partial`。
没有任何 full match 时不得强行选择最高 Fit，应输出 `official_registry_coverage_gap`。

### Step A8 — Qualification / Status Gate

资质只有在当前需求或企业正式规则明确要求时作为 Hard Gate。
资质过期触发人工核验，不扩大为法律结论。
官方库没有明确有效/冻结/禁用状态时：`registry_status = unknown`。

### Step A9 — Hard Gate Result

每家供方至少记录：official_source、registry_tier、material_supplier、category、region、qualification、registry_status、evidence。

总体结果：

- `eligible_for_review`
- `partial_match_for_review`
- `needs_human_check`
- `ineligible`

### Step A10 — Fit Analysis

只对通过或部分通过 Hard Gate 的供方做辅助分析。Fit 不得绕过 Hard Gate。

### Step A11 — Candidate Pool

按以下分组：

- Full Match
- Partial Match
- Needs Official Data Check
- Excluded

每家供方必须给出官方 evidence。

### Step A12 — Coverage Gap Check

检查：

- 品类/SKU 是否无官方供方覆盖；
- 区域是否无完整候选；
- 是否只有单一候选导致竞争不足；
- 当前供方库是否只是局部区域导出；
- 关键官方字段是否普遍缺失或过旧。

只能请求更多官方数据，不能转向公网补供方。

### Step A13 — Human Supplier Checkpoint（强制暂停）

采购员决定：

- `include_for_intention_collection`
- `exclude`
- `hold`
- `request_more_official_data`

未得到人工确认前：

- 不得生成正式 BCC 邀标邮件；
- 不得生成正式供方信息长名单；
- 不得把候选称为“已确定邀标供应商”；
- 不得进入 Phase B。

人工确认后形成 `confirmed_initial_supplier_pool`。

## 7. Workflow — Phase B Invitation Package

### Step B1 — Validate Initial Supplier Pool

04 长名单和 BCC 都必须严格以 `confirmed_initial_supplier_pool` 为边界，不得新增未确认供方。

### Step B2 — Generate Supplier Contact Longlist（强制）

使用：

`.agents/skills/sourcing-invitation/templates/供方信息长名单模板.xlsx`

生成：

`{{项目名称}}-供方信息长名单.xlsx`

固定字段：

- 序号
- 供方名称
- 联系人姓名
- 联系电话
- 邮箱地址
- 联系信息来源
- 联系信息状态
- 备注

**不输出“供方编码”字段。** 供方内部 ID / supplier_id 可继续存在于结构化数据中用于追溯，但不得作为 04 长名单展示列。

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
冲突时：`BLOCK/HOLD: supplier_contact_bcc_mismatch`。

### Step B4 — Generate Single EML

生成：`{{项目名称}}-邀标邮件.eml`

基础规则：

- 每个项目只生成 1 个 EML；
- Bcc = 全部已确认且邮箱已确认的供方；
- To 仅可使用采购方明确提供的本方发送/归档邮箱，否则留空；
- Cc 仅使用采购员明确确认的内部邮箱；
- 正文只引用 02 与 03；
- 不引用 04；
- EML 不嵌入附件；
- 不自动发送。

#### Markdown-Free EML Gate（强制）

最终 `.eml` 正文必须为可直接阅读的纯文本或标准 HTML：

- 推荐 `text/plain; charset=utf-8`；
- 如使用 HTML，则必须是标准 `text/html; charset=utf-8`；
- 内部 `.md` 模板只允许作为变量源，不得原样写入 EML；
- 写入 EML 前必须移除/转换 Markdown 标题、强调、列表、引用、代码、链接等格式标记；
- 纯文本分点优先使用中文序号、数字序号或 `•`，不要使用 Markdown 的 `- ` / `* ` / `+ ` 列表符；
- 校验对象是“解码后的邮件正文”，不扫描 MIME Header、文件名或编码字节。

以下 Markdown 格式标记不得残留在最终正文：

- 行首 `# `、`## ` 等标题标记；
- `**`、`__` 等强调标记；
- 反引号和代码围栏；
- `[文字](链接)` Markdown 链接；
- 行首 `> ` 引用标记；
- 行首 `- `、`* `、`+ ` 无序列表标记。

发现任一 Markdown 格式残留时：

`BLOCK/HOLD: eml_markdown_marker_detected`

不得标记为 `ready_for_human_review`，必须清洗后重新生成。

详细规则见：

- `references/eml-delivery-rules.md`
- `references/eml-markdown-free-test.md`

### Step B5 — Prepare Final Requirement Workbook

02 = 人工确认的最终需求清单。
不得重新优化、改数量、改规格、改区域或预填供方报价字段。

### Step B6 — Calculate Bid Bond Variable

读取 `procurement_estimated_total_amount`。

计算：

- `raw_bid_bond = procurement_estimated_total_amount × 1%`
- `bid_bond_amount = CEILING(raw_bid_bond, 1000)`

即向上取整到 1000 元整数倍。

预估总金额必须来自可追溯的当前项目数据，禁止从供方待填写报价列推断，也禁止沿用历史固定保证金。

详细规则见：`references/bid-bond-variable-rules.md`。

### Step B7 — Prepare Intention Form from Original Enterprise Template

03 必须复制：

`.agents/skills/sourcing-invitation/templates/招标意向征集登记表模板.xlsx`

流程：

`copy original template → populate allowed project variables → replace N3 variable → save project file`

`Sheet1!N3`：`{{投标保证金金额}} → bid_bond_amount`。

必须保持原模板工作表、行列结构、合并单元格、字体、边框、填充、对齐、行高列宽、打印设置、公式/验证及供方填写区域。
禁止重新设计或新建类似表格替代企业原模板。

### Step B8 — Consistency Check

至少检查：

- 项目名称
- 配送区域
- 商务条件
- 合作期限
- 联系人/回复信息
- 采购清单预估总金额
- 投标保证金
- 04 长名单供方集合 = 人工确认初版供方集合
- BCC 均能回溯到 04 长名单
- 02/03 是唯一对外 Excel 附件
- EML 无嵌入附件
- EML 解码后正文通过 Markdown-Free Gate

### Step B9 — Human Send Review

Skill 只生成邮件草稿和文件，不自动发送。

发送前由采购员审核：

- BCC
- To/Cc
- 邮件正文是否显示为正常纯文本/HTML且无 Markdown 标记
- 02/03 附件
- 03 中投标保证金金额
- 截止时间及其他当前项目变量

## 8. Fixed Deliverables

Phase B 固定交付：

1. `{{项目名称}}-邀标邮件.eml`
2. `{{项目名称}}-最终需求清单.xlsx`
3. `{{项目名称}}-招标意向征集登记表.xlsx`
4. `{{项目名称}}-供方信息长名单.xlsx`

其中 04 为采购内部文件，不对外发送。

## 9. Structured Handoff

Phase A 人工确认后应保留 `confirmed_initial_supplier_pool`。

Phase B 完成后输出结构化状态，至少包含：

```yaml
handoff:
  workflow_stage: invitation_package_ready
  supplier_source: official_registry_only
  confirmed_initial_supplier_pool: []
  supplier_contact_longlist:
    generated: true
    filename: "{{项目名称}}-供方信息长名单.xlsx"
    displayed_fields:
      - 序号
      - 供方名称
      - 联系人姓名
      - 联系电话
      - 邮箱地址
      - 联系信息来源
      - 联系信息状态
      - 备注
  invitation_email:
    generated: true
    format: eml
    mode: single_bcc_email
    body_format: plain_text_or_html
    markdown_free: true
    auto_send: false
  bid_bond:
    rate: 0.01
    rounding_unit: 1000
    rounding_mode: ceiling
  artifacts:
    external_attachments:
      - "{{项目名称}}-最终需求清单.xlsx"
      - "{{项目名称}}-招标意向征集登记表.xlsx"
    internal_files:
      - "{{项目名称}}-供方信息长名单.xlsx"
  next_skill: supplier-shortlist
```

## 10. Guardrails

- 供方身份只能来自企业官方供方库。
- 公网不得新增邀标候选供方。
- Phase A 候选池不得等同于正式邀标名单。
- 未经采购员确认不得进入 Phase B。
- 不猜联系人、电话、邮箱、资质、区域能力。
- 04 长名单不展示供方编码。
- BCC 必须回溯到 04 长名单。
- EML 不嵌入附件且不自动发送。
- EML 最终正文不得保留 Markdown 格式标记。
- 02/03 单独交付，由采购员发送前人工添加。
- 03 必须复制企业原模板生成。
- N3 保证金按当前项目预估总金额 × 1%，千元位向上取整。
- 缺少可追溯预估总金额时不得默认保证金。

## 11. Success Criteria

1. 官方库匹配与人工确认边界清楚；
2. 候选供方均有官方 evidence；
3. Phase B 只覆盖人工确认初版供方；
4. 04 长名单字段与模板一致且不包含供方编码；
5. BCC 与长名单邮箱一致；
6. EML 解码后正文是正常纯文本/HTML且 Markdown-Free；
7. 03 使用企业原模板且 N3 动态保证金正确；
8. 邀标四件套可以直接供采购员审核使用；
9. `supplier-shortlist` 可直接消费确认供方及后续真实回复。
