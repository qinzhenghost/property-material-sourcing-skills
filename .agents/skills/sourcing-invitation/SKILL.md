---
name: sourcing-invitation
description: 在采购员确认官方库候选供方后，基于已确认的物业物资需求、已确认供方邮箱、企业邀标邮件模板和招标意向征集登记表模板，一次生成并校验邀标沟通三件套：单一 BCC 群发邀标邮件草稿 .eml、最终需求清单.xlsx、招标意向征集登记表.xlsx。EML 不嵌入附件，不自动发送。
metadata:
  version: "0.3.1"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
---

# Sourcing Invitation

## 1. Purpose

将已经确认的采购需求和人工确认的官方库候选供方转换为可直接供采购员审核的标准邀标沟通包。

本 Skill 固定输出三个独立交付物：

1. `{{项目名称}}-邀标邮件.eml`
2. `{{项目名称}}-最终需求清单.xlsx`
3. `{{项目名称}}-招标意向征集登记表.xlsx`

其中：

- 只生成 **1 个 EML**；
- 所有人工确认参与邀标且邮箱已确认的供方邮箱统一写入 `Bcc`；
- 不为每家供方单独生成 EML；
- EML **不嵌入任何附件**；
- 两个 Excel 作为独立文件交付，由采购员审核后在实际发送时添加；
- 不自动发送邮件。

## 2. Required Upstream

必须具备：

- `material-requirement-analysis` 的 confirmed requirement / 最终需求清单；
- `official-supplier-matching` 经采购员人工确认的邀标沟通供方；
- 企业现行邀请邮件模板；
- 企业现行《招标意向征集登记表》模板；
- 当前项目联系人；
- 回复截止时间。

供方邮箱可以来自：

1. 企业官方供方库明确字段；
2. 采购员人工补充并确认的邮箱。

不得从公司名称、联系人姓名、网站域名或公网搜索猜测邮箱。

## 3. Source of Truth

优先级：

1. 人工确认的标准需求；
2. 人工确认的候选供方名单；
3. 人工确认/官方库中的供方邮箱；
4. 当前项目明确的商务/流程条件；
5. 企业邮件和附件模板。

模板只是容器。模板中的历史日期、地区、账期、保证金、联系人等不得自动继承到当前项目。

## 4. Workflow

### Step 1 — Validate Human Checkpoint

确认上游候选供方已经由采购员确认。

未确认：

`BLOCK: supplier list not human-confirmed`

### Step 2 — Build Recipient Set

对人工确认参与邀标沟通的供方逐一读取：

- supplier_id
- supplier_name
- contact_name（如有）
- contact_email
- email_source

邮箱规则：

- 邮箱明确且来源可追溯 → `confirmed_recipient_email`
- 邮箱缺失 → `missing_recipient_email`
- 邮箱存在但来源不明确/有冲突 → `needs_email_confirmation`

生成：

```yaml
bcc_recipients:
  - supplier_name: 示例供方A
    email: a@example.com
    source: official_registry
missing_recipient_emails:
  - 示例供方B
```

**不得因为个别供方缺邮箱而猜测邮箱。**

如存在缺邮箱供方，可以生成 EML 草稿，但 `email_status = draft_missing_recipient_email`，并明确提醒采购员补充后再发送。

### Step 3 — Build Project Variable Set

至少抽取：

- project_name
- delivery_regions
- cooperation_period
- payment_terms
- quotation_basis
- pricing_instruction
- framework_and_ordering_terms
- contact_name
- contact_phone
- contact_email
- response_deadline
- requirement_filename
- intention_form_filename

### Step 4 — Generate Single EML

使用 `templates/invitation-email-template.md` 生成邮件正文，并封装为 RFC 5322 / MIME 兼容 `.eml`。

字段规则：

- `Subject`：当前项目邀标/意向征集主题；
- `Bcc`：全部 `confirmed_recipient_email`；
- `To`：不得填写任何供方邮箱。若采购员提供本方发送/归档邮箱，可写入；否则保持空白并标记待发送前确认；
- `Cc`：仅使用采购员明确提供的内部抄送邮箱；不得猜测；
- `From`：仅在当前运行环境/用户明确提供发件邮箱时填写；否则留空；
- 正文：同一份通用邀标正文，不按供方分别个性化；
- `attachments_embedded = false`。

邮件正文应明确：

- 项目名称；
- 配送范围；
- 当前邀标/意向征集目的；
- `{{项目名称}}-最终需求清单.xlsx` 的用途；
- `{{项目名称}}-招标意向征集登记表.xlsx` 的填写/回传要求；
- 报价口径或当前阶段是否需要报价；
- 合作/平台/订单规则（若适用）；
- 回复截止时间；
- 项目联系人及回复邮箱。

**EML 只引用附件文件名，不把两个 Excel 编码进 MIME 附件。**

详细规则见 `references/eml-delivery-rules.md`。

### Step 5 — Prepare Final Requirement Workbook

输出：

`{{项目名称}}-最终需求清单.xlsx`

必须来自人工确认的最终需求，不得重新“优化”采购需求。

允许：

- 复制上游最终需求清单；
- 清理 AI 内部诊断备注；
- 保留供方报价字段为空；
- 更新当前项目已确认说明。

### Step 6 — Prepare Supplier Intention Form

输出：

`{{项目名称}}-招标意向征集登记表.xlsx`

基于企业现行模板。

默认保留供方填写项为空。不得替供方回答：

- 是否有合作意向；
- 是否接受条款；
- 服务/供货能力；
- 资质声明；
- 报价或承诺。

### Step 7 — Three-Artifact Consistency Check

检查 EML、最终需求清单、意向征集登记表之间：

- 项目名称；
- 配送区域；
- 账期；
- 合作期限；
- 平台/下单规则；
- 报价口径；
- 联系人；
- 回复邮箱；
- 截止时间；
- 正文引用的两个附件文件名。

同时检查：

- BCC 是否仅包含人工确认供方；
- BCC 邮箱是否有明确来源；
- 是否有供方邮箱被误放入 To/Cc；
- EML 是否未嵌入附件。

### Step 8 — Output Package

固定输出：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
```

并输出 `schemas/sourcing-invitation-package.schema.yaml` 对应 manifest。

不得额外为每家供方生成独立 EML。

### Step 9 — Human Send Checkpoint

生成完成 ≠ 自动发送。

采购员发送前必须确认：

- BCC 供方名单与邮箱；
- 缺失/待确认邮箱是否已经补齐；
- To / Cc / From；
- 邮件标题和正文；
- 两个独立 Excel 文件；
- 实际发送时两个 Excel 是否均已添加为附件；
- 回复截止时间和商务条件。

## 5. Output Rules

### EML

- 只生成一个；
- 供方统一进入 `Bcc`；
- 供方不得出现在 `To` 或 `Cc`；
- 不嵌入 Excel；
- 不自动发送；
- 邮箱缺失时不得猜测。

### Final Requirement Workbook

必须与 confirmed requirement 等价。

### Supplier Intention Form

必须使用企业现行模板并保持字段结构。

## 6. Guardrails

AI MUST NOT：

- 新增官方库外供方；
- 猜测供方邮箱；
- 将不同供方放在 To/Cc 导致互相可见；
- 为每家供方默认生成独立 EML；
- 把两个 Excel 嵌入 EML；
- 替供方填写意愿、能力或报价；
- 改变确认过的账期、配送范围、保证金、合作周期、报价口径；
- 沿用历史模板中的项目变量；
- 自动发送邮件。

## 7. Review Checklist

- [ ] 候选供方是否已人工确认？
- [ ] BCC 是否只包含已确认供方？
- [ ] 每个 BCC 邮箱是否来自官方库或人工确认？
- [ ] 是否没有供方邮箱出现在 To/Cc？
- [ ] 是否只生成一个 EML？
- [ ] EML 是否未嵌入附件？
- [ ] 正文是否准确引用两个独立 Excel 文件名？
- [ ] 最终需求清单是否来自 confirmed requirement？
- [ ] 意向征集表是否为企业现行模板？
- [ ] 是否没有替供方填写响应字段？
- [ ] 三件套关键项目变量是否一致？
- [ ] 是否停在人工发送确认节点？

## 8. Success Criteria

1. 一次运行稳定生成一个 `.eml` + 两个独立 `.xlsx`；
2. EML 的 BCC 覆盖全部已确认且邮箱已确认的供方；
3. 缺邮箱供方被明确提示而非猜测；
4. 供方邮箱不会彼此可见；
5. EML 不嵌入附件；
6. 两个 Excel 可独立审核和交付；
7. 三件套内容一致；
8. 输出可直接供采购员审核后发送。
