---
name: sourcing-invitation
description: 在采购员确认官方库候选供方后，基于已确认的物业物资需求、现有邀标邮件模板和招标意向征集登记表模板，一次生成并校验邀标沟通三件套：邮件正文、最终需求清单、招标意向征集登记表。V1适用于物业物资类邀标制采购。
metadata:
  version: "0.3.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
---

# Sourcing Invitation

## 1. Purpose

将已经确认的采购需求和官方库候选供方转换为可直接供采购员审核、发送的标准邀标沟通包。

本 Skill 的标准输出**只有三个产出物**：

1. 邀标/意向征集邮件正文；
2. 最终需求清单（附件1）；
3. 招标意向征集登记表（附件2）。

不设置独立的 `supplier-intention-screening` Skill。

供方收到邮件并回传附件2后的信息，作为后续短名单/回标分析输入。

## 2. Required Upstream

必须同时具备：

- `material-requirement-analysis` 的 confirmed requirement；
- `official-supplier-matching` 的人工确认候选供方；
- 企业现行邀请邮件模板；
- 企业现行《招标意向征集登记表》模板；
- 项目联系人和回复截止时间。

任一关键输入缺失时，不得生成可发送版本。

## 3. Source of Truth

优先级：

1. 人工确认的标准需求；
2. 人工确认的候选供方名单；
3. 当前项目明确的商务/流程条件；
4. 企业邮件和附件模板。

模板只是容器。

如果模板中的历史项目内容与当前确认需求冲突，必须以当前确认需求为准，并提示采购员发生了模板变量替换。

## 4. Workflow

### Step 1 — Validate Human Checkpoint

确认上游候选供方已经由采购员确认。

未确认：

`BLOCK: supplier list not human-confirmed`

### Step 2 — Build Project Variable Set

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

禁止从历史模板中继承当前项目未确认的日期、地区、保证金、账期等变量。

### Step 3 — Generate Email Body

使用 `templates/invitation-email-template.md`。

邮件应明确：

- 邀请参加的项目；
- 附件1用途；
- 需求澄清方式；
- 框架/采购平台/订单模式（若项目适用）；
- 报价口径或当前阶段“不回应具体报价”的规则；
- 配送范围；
- 附件2填写要求；
- 无意向时的回复要求；
- 回复截止时间；
- 联系人和邮箱。

### Step 4 — Prepare Attachment 1

附件1 = 人工确认的最终需求清单。

不得重新“优化”需求。

只允许：

- 将结构化 requirement 渲染回标准 Excel 模板；
- 清理 AI 诊断过程中的内部备注；
- 保留供方应填写的报价字段为空；
- 根据项目确认结果更新项目级说明。

### Step 5 — Prepare Attachment 2

附件2 = `templates/招标意向征集登记表模板.xlsx`。

默认保留供方填写项为空。

只有在同时满足以下条件时可预填公司名称/联系人：

- 数据来自官方供方库；
- 字段明确；
- 采购员允许预填。

不得替供方回答任何意愿、能力、条款接受度或资质声明。

### Step 6 — Three-Artifact Consistency Check

逐项检查邮件、附件1、附件2的一致性：

- 项目名称
- 配送区域
- 账期
- 合作期限
- 平台下单规则
- 报价阶段/报价口径
- 保证金要求
- 联系人
- 邮箱
- 截止时间

发现冲突：

- 不得静默修正关键条款；
- 输出冲突字段；
- 以 confirmed requirement 为基准提出修正；
- 等待采购员确认。

### Step 7 — Output Package

固定输出：

```text
01 邀标邮件正文.md
02 最终需求清单.xlsx
03 招标意向征集登记表.xlsx
```

并输出 `sourcing-invitation-package.schema.yaml` 对应的结构化 manifest。

### Step 8 — Human Send Checkpoint

生成完成 ≠ 自动发送。

采购员必须最终确认：

- 收件供方；
- 邮件正文；
- 两个附件；
- 截止时间；
- 关键商务条款。

只有采购员明确要求发送，运行环境又具备邮件工具时，才可进入发送动作。

## 5. Output Rules

### Email

生成一份标准邮件正文，可根据收件供方批量复用。

若需要个性化，只允许使用官方库中的：

- 供方名称；
- 联系人姓名。

不得个性化修改商务条件。

### Requirement List

必须与 confirmed requirement 等价。

### Intention Form

必须使用企业模板并保持字段结构。

## 6. Guardrails

- 不新增官方库外供方。
- 不替供方填写意愿或能力声明。
- 不把“意向征集表”当作 AI 自己的资格判断表。
- 不生成第四个默认附件。
- 不自行改变账期、配送范围、保证金、合作周期、报价口径。
- 不将历史模板日期和联系人直接沿用到新项目。
- 不把当前“前期需求沟通”误写成“正式报价”，除非项目规则明确。
- 不自动发送邮件。

## 7. Review Checklist

- [ ] 候选供方是否已人工确认？
- [ ] 邮件项目名称是否为当前项目？
- [ ] 邮件是否正确引用附件1和附件2？
- [ ] 附件1是否来自 confirmed requirement？
- [ ] 附件2是否为现行意向征集表模板？
- [ ] 是否没有替供方填写意向/能力字段？
- [ ] 配送区域三处是否一致？
- [ ] 账期/合作期限/平台规则是否一致？
- [ ] 回复截止时间是否明确？
- [ ] 联系人电话和邮箱是否来自当前项目输入？
- [ ] 是否只生成三个标准产出物？
- [ ] 是否停在人工发送确认节点？

## 8. Success Criteria

1. 一次运行稳定生成三件套；
2. 三件套关键项目变量一致；
3. 附件1不篡改确认需求；
4. 附件2不代替供方作答；
5. 邮件内容与企业现有邀请风格和流程保持一致；
6. 输出可直接供采购员审核发送。
