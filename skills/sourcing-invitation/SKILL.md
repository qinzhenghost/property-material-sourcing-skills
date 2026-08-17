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

生成 `bcc_recipients` 和 `missing_recipient_emails`。

不得因为个别供方缺邮箱而猜测邮箱。如存在缺邮箱供方，可以生成 EML 草稿，但 `email_status = draft_missing_recipient_email`，并提醒采购员补充后再发送。

### Step 3 — Build Project Variable Set

抽取项目名称、配送区域、合作周期、账期、报价口径、联系人、回复邮箱、截止时间和两个附件文件名。

### Step 4 — Generate Single EML

使用 `templates/invitation-email-template.md` 生成邮件正文，并封装为 RFC 5322 / MIME 兼容 `.eml`。

字段规则：

- `Subject`：当前项目邀标/意向征集主题；
- `Bcc`：全部 `confirmed_recipient_email`；
- `To`：不得填写任何供方邮箱。若采购员提供本方发送/归档邮箱，可写入；否则保持空白并标记待发送前确认；
- `Cc`：仅使用采购员明确提供的内部抄送邮箱；
- `From`：仅在明确提供时填写；
- 正文：同一份通用邀标正文；
- `attachments_embedded = false`。

邮件正文必须准确引用：

- `{{项目名称}}-最终需求清单.xlsx`
- `{{项目名称}}-招标意向征集登记表.xlsx`

EML 只引用附件文件名，不把两个 Excel 编码进 MIME 附件。

详细规则见 `references/eml-delivery-rules.md`。

### Step 5 — Prepare Final Requirement Workbook

输出 `{{项目名称}}-最终需求清单.xlsx`，来源必须是人工确认需求，不得重新优化采购需求。

### Step 6 — Prepare Supplier Intention Form

输出 `{{项目名称}}-招标意向征集登记表.xlsx`，基于企业现行模板，供方填写项保持空白。

### Step 7 — Three-Artifact Consistency Check

检查项目名称、配送区域、账期、合作期限、报价口径、联系人、回复邮箱、截止时间及附件文件名一致性；同时检查 BCC 仅包含确认供方、邮箱可追溯、供方未出现在 To/Cc、EML 未嵌入附件。

### Step 8 — Output Package

固定输出：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
```

并输出 `schemas/sourcing-invitation-package.schema.yaml` 对应 manifest。

### Step 9 — Human Send Checkpoint

采购员发送前必须确认 BCC、缺失邮箱、To/Cc/From、正文、两个独立 Excel、实际发送时已添加两个附件、截止时间和商务条件。

## 5. Guardrails

- 不新增官方库外供方。
- 不猜测供方邮箱。
- 不将供方放入 To/Cc。
- 不为每家供方生成独立 EML。
- 不把 Excel 嵌入 EML。
- 不替供方填写意愿、能力、报价。
- 不改变确认过的商务条件。
- 不沿用历史模板项目变量。
- 不自动发送邮件。

## 6. Success Criteria

1. 一次运行生成一个 `.eml` + 两个独立 `.xlsx`；
2. BCC 覆盖全部已确认且邮箱已确认的供方；
3. 缺邮箱供方明确提示而非猜测；
4. 供方邮箱不会彼此可见；
5. EML 不嵌入附件；
6. 两个 Excel 独立交付；
7. 三件套内容一致；
8. 输出可直接供采购员审核后发送。
