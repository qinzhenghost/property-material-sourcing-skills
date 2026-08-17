---
name: sourcing-invitation
description: 在采购员确认官方库候选供方后，基于已确认的物业物资需求、已确认供方邮箱、企业原版邀标邮件模板和企业原版招标意向征集登记表模板，一次生成并校验邀标沟通三件套：单一 BCC 群发邀标邮件草稿 .eml、最终需求清单.xlsx、基于企业原版模板复制生成的招标意向征集登记表.xlsx。登记表 N3 投标保证金金额按采购清单预估总金额的1%计算并向上取整至千元整数倍。EML 不嵌入附件，不自动发送。
metadata:
  version: "0.3.3"
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

- 只生成 1 个 EML；
- 所有人工确认参与邀标且邮箱已确认的供方邮箱统一写入 `Bcc`；
- EML 不嵌入任何附件；
- 两个 Excel 独立交付；
- 不自动发送邮件；
- 03 登记表必须以企业原始模板文件为底稿复制生成；
- 03 登记表 `Sheet1!N3` 中的投标保证金金额是项目变量，不允许继续使用历史固定金额。

## 2. Required Upstream

必须具备：

- `material-requirement-analysis` 的 confirmed requirement / 最终需求清单；
- `official-supplier-matching` 的人工确认候选供方；
- 已确认的供方邮箱，或可明确指出缺失邮箱；
- 企业现行邀请邮件模板；
- 企业原版《招标意向征集登记表》模板；
- 项目联系人和回复截止时间；
- 可追溯的 `采购清单预估总金额`，用于计算投标保证金。

如果除预估总金额外的关键输入缺失，不得生成可发送版本。

如果仅缺少 `采购清单预估总金额`：

- EML 与最终需求清单可先形成草稿；
- 03 登记表不得标记为 ready；
- 必须提示采购员补充/确认该金额后再完成 N3 变量替换。

## 3. Source of Truth

优先级：

1. 人工确认的标准需求；
2. 人工确认的候选供方名单；
3. 当前项目明确的商务/流程条件；
4. 当前项目可追溯的采购清单预估总金额；
5. 企业原版邮件模板；
6. 企业原版《招标意向征集登记表》模板。

模板是企业正式载体。不得把企业模板替换为 AI 自行设计的表格。

### 采购清单预估总金额来源

按以下优先级读取：

1. 当前最终需求/采购清单中已经人工确认的预估总金额；
2. 上游 `historical-procurement-analysis` / `material-requirement-analysis` 已形成且可追溯的项目预估总金额；
3. 采购员明确确认的本项目预估总金额。

不得：

- 从供应商尚未填写的报价列计算；
- 沿用历史模板中的 6000 元或其他固定金额；
- 自行猜测单价或预估总金额。

## 4. Workflow

### Step 1 — Validate Human Checkpoint

确认上游候选供方已经由采购员确认。

未确认：

`BLOCK: supplier list not human-confirmed`

### Step 2 — Build Recipient Set

从人工确认候选供方中读取官方/采购员确认邮箱。

- 已确认邮箱 → 写入 Bcc；
- 缺失/冲突邮箱 → 写入 `missing_recipient_emails`；
- 不猜测邮箱；
- 供方邮箱不得进入 To/Cc。

### Step 3 — Generate Single EML

生成：

`{{项目名称}}-邀标邮件.eml`

规则：

- 单一 EML；
- Bcc = 全部已确认供方邮箱；
- To 仅可填采购方明确提供的本方发送/归档邮箱，否则留空；
- Cc 仅可填明确确认的内部抄送邮箱；
- 正文引用附件1/附件2文件名；
- 不嵌入附件；
- 不自动发送。

### Step 4 — Prepare Attachment 1

附件1 = 人工确认的最终需求清单。

不得重新优化或改变 confirmed requirement。

### Step 5 — Calculate Bid Bond Variable（强制）

读取 `采购清单预估总金额`：

`procurement_estimated_total_amount`

计算：

`raw_bid_bond = procurement_estimated_total_amount × 1%`

`bid_bond_amount = CEILING(raw_bid_bond, 1000)`

即：**向上取整到 1,000 元的整数倍。**

示例：

- 523,673.81 元 × 1% = 5,236.7381 元 → 6,000 元；
- 1,234,567 元 × 1% = 12,345.67 元 → 13,000 元；
- 80,000 元 × 1% = 800 元 → 1,000 元。

计算结果必须保留：

- 采购清单预估总金额；
- 保证金比例 `1%`；
- 取整单位 `1000`；
- 取整方式 `ceiling`；
- 最终投标保证金金额；
- 金额数据来源。

详细规则见 `references/bid-bond-variable-rules.md`。

### Step 6 — Prepare Attachment 2 from Original Enterprise Template（强制）

附件2必须从仓库内企业原版模板：

`.agents/skills/sourcing-invitation/templates/招标意向征集登记表模板.xlsx`

按文件副本方式生成：

`copy original template → populate allowed current-project fields → replace N3 variable → save as {{项目名称}}-招标意向征集登记表.xlsx`

企业模板 `Sheet1!N3` 必须包含变量：

`{{投标保证金金额}}`

生成项目文件时，将其替换为 Step 5 的 `bid_bond_amount`。

例如：

`是否愿意\n开标前缴纳预计{{投标保证金金额}}元投标保证金……`

在项目预估总金额为 523,673.81 元时应生成：

`是否愿意\n开标前缴纳预计6000元投标保证金……`

除变量替换外，必须保持原模板：

- 工作表数量与名称；
- 行列结构；
- 合并单元格；
- 字体/字号/颜色；
- 边框、填充、对齐；
- 行高、列宽；
- 打印设置；
- 公式与数据验证（如原模板存在）；
- 供方填写区域及其位置；
- 企业模板已有说明文字与固定字段结构。

禁止：

- 新建一个“类似”的登记表代替原模板；
- 用通用标准模板重新渲染；
- 调整版式使其“更美观”；
- 删除企业模板原有字段；
- 改变供方填写区布局；
- 将其他项目/历史案例表作为模板；
- 在 N3 使用历史固定金额；
- 在预估总金额缺失时默认写 0 元、6000 元或任意金额。

供方填写字段必须保持空白；除非采购员明确要求且字段来自官方供方库，不预填供应商公司名称或联系人。

### Step 7 — Original Template & Bid Bond Integrity Check（强制）

生成 03 文件后检查：

1. 输出是从企业原模板复制而来，而非新建 workbook；
2. 原模板结构和样式未被无关修改；
3. 仅允许的项目变量发生变化；
4. 供方待填写字段仍为空；
5. `Sheet1!N3` 不再包含 `{{投标保证金金额}}`；
6. N3 金额 = `CEILING(采购清单预估总金额 × 1%, 1000)`；
7. 文件可正常打开且无公式错误。

如果不能保证模板完整性：

`BLOCK: original intention form template integrity not preserved`

如果无法获得采购清单预估总金额：

`BLOCK: missing_procurement_estimated_total_amount`

如果 N3 金额计算或替换不一致：

`BLOCK: bid_bond_variable_mismatch`

### Step 8 — Three-Artifact Consistency Check

检查邮件、最终需求清单、意向征集登记表：

- 项目名称；
- 配送区域；
- 账期；
- 合作期限；
- 平台/订单规则；
- 回复截止时间；
- 联系人；
- 邮箱；
- 两个附件文件名；
- 采购清单预估总金额；
- 投标保证金金额。

### Step 9 — Output Package

固定输出：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
```

并输出结构化 manifest。

### Step 10 — Human Send Checkpoint

生成完成不等于自动发送。采购员最终确认 Bcc、正文、两个独立附件、截止时间、商务条款和投标保证金金额。

## 5. Guardrails

- 官方供方库 ONLY；
- 不猜供方邮箱；
- 供方邮箱仅进入 Bcc；
- 不自动发送；
- EML 不嵌入附件；
- 不改变 confirmed requirement；
- 不替供方填写意愿/能力；
- 不重建、重新设计或替换企业原版招标意向征集登记表模板；
- 不从历史项目模板继承未确认变量；
- 不把历史固定保证金金额作为当前项目保证金；
- 不从供应商报价空白列反推采购清单预估总金额；
- 保证金固定按当前规则 `1% + 千元向上取整` 计算，除非采购员明确修改当前项目规则。

## 6. Success Criteria

1. 仅生成一个 BCC EML；
2. 两个 Excel 独立交付；
3. 03 登记表与企业原始模板结构/版式一致；
4. `Sheet1!N3` 的保证金金额来自当前项目预估总金额动态计算；
5. N3 金额满足 `CEILING(采购清单预估总金额 × 1%, 1000)`；
6. 供方填写区保持空白；
7. 三件套关键项目变量一致；
8. 停在人工发送确认节点。
