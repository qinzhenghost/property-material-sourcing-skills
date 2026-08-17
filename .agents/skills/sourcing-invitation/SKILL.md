---
name: sourcing-invitation
description: 在采购员确认官方库初版供方后，基于已确认的物业物资需求、供方联系信息、企业原版邀标邮件模板和企业原版招标意向征集登记表模板，生成并校验邀标沟通四件套：单一 BCC 群发邀标邮件草稿 .eml、最终需求清单.xlsx、基于企业原版模板复制生成的招标意向征集登记表.xlsx、采购内部使用的供方信息长名单.xlsx。登记表 N3 投标保证金金额按采购清单预估总金额的1%计算并向上取整至千元整数倍。EML 不嵌入附件，不自动发送。
metadata:
  version: "0.3.4"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
---

# Sourcing Invitation

## 1. Purpose

将已经确认的采购需求和人工确认的官方库初版供方池转换为可直接供采购员审核的标准邀标沟通包。

本 Skill 固定输出四个独立交付物：

1. `{{项目名称}}-邀标邮件.eml`
2. `{{项目名称}}-最终需求清单.xlsx`
3. `{{项目名称}}-招标意向征集登记表.xlsx`
4. `{{项目名称}}-供方信息长名单.xlsx`

其中：

- 只生成 1 个 EML；
- 所有人工确认参与邀标且邮箱已确认的供方邮箱统一写入 `Bcc`；
- EML 不嵌入任何附件；
- 对外实际发送附件仍只有 02、03；
- 04 供方信息长名单是采购内部文件，不得发送给供方，也不得在对外 EML 正文中引用；
- 03 登记表必须以企业原始模板文件为底稿复制生成；
- 03 登记表 `Sheet1!N3` 中的投标保证金金额是项目变量，不允许继续使用历史固定金额；
- 不自动发送邮件。

## 2. Required Upstream

必须具备：

- `material-requirement-analysis` 的 confirmed requirement / 最终需求清单；
- `official-supplier-matching` 的采购员人工确认初版供方池；
- 初版供方的联系人/电话/邮箱官方字段或人工确认信息（允许部分缺失）；
- 企业现行邀请邮件模板；
- 企业原版《招标意向征集登记表》模板；
- 项目联系人和回复截止时间；
- 可追溯的 `采购清单预估总金额`，用于计算投标保证金。

联系信息允许缺失，但不得猜测。缺失内容必须在 04 长名单和 manifest 中显式标记。

如果仅缺少 `采购清单预估总金额`：

- EML、最终需求清单和供方信息长名单可先形成草稿；
- 03 登记表不得标记为 ready；
- 必须提示采购员补充/确认该金额后再完成 N3 变量替换。

## 3. Source of Truth

优先级：

1. 人工确认的标准需求；
2. 人工确认的初版供方名单；
3. 企业官方供方库中的联系人、电话、邮箱字段；
4. 采购员明确确认/补充的供方联系信息；
5. 当前项目明确的商务/流程条件；
6. 当前项目可追溯的采购清单预估总金额；
7. 企业原版邮件模板；
8. 企业原版《招标意向征集登记表》模板。

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

### 供方联系信息来源

联系人姓名、联系电话、邮箱地址只允许来自：

1. 企业官方供方库明确字段；
2. 采购员明确确认/补充的信息。

不得根据公司名称、联系人姓名、邮箱域名规律、电话号码规律或公网信息猜测/新增联系信息。

## 4. Workflow

### Step 1 — Validate Human Checkpoint

确认上游初版供方池已经由采购员确认。

未确认：

`BLOCK: supplier list not human-confirmed`

### Step 2 — Build Confirmed Initial Supplier Pool

固定本次输出供方范围：

`confirmed_initial_supplier_pool`

要求：

- 04 长名单必须覆盖该集合中的全部供方；
- 不得只保留有邮箱的供方；
- 不得新增未确认供方；
- 不得从公网新增供方。

### Step 3 — Generate Supplier Contact Longlist（强制）

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

1. 长名单覆盖全部人工确认初版供方；
2. 联系人/电话/邮箱只读取官方库或人工确认信息；
3. 缺失字段保持空白并标记状态，不得猜测；
4. 同一供方存在多个已确认联系人时，可一条联系人记录一行，重复供方名称；
5. 建议状态：`完整 / 缺联系人 / 缺电话 / 缺邮箱 / 多项缺失 / 待人工确认`；
6. 04 是内部文件，不得作为供方邮件附件或在对外正文中引用。

详细规则见 `references/supplier-contact-longlist-rules.md`。

### Step 4 — Build Recipient Set from Longlist

从 04 长名单中读取已确认邮箱：

- 已确认邮箱 → 写入 Bcc；
- 缺失/冲突邮箱 → 写入 `missing_recipient_emails`；
- 不猜测邮箱；
- 供方邮箱不得进入 To/Cc。

一致性要求：

- 每个 BCC 邮箱必须能回溯到 04 长名单中的已确认供方/联系人；
- 04 长名单中缺邮箱的供方不得被凭空加入 BCC；
- 04 中的邮箱与 BCC 冲突时：`BLOCK/HOLD: supplier_contact_bcc_mismatch`。

### Step 5 — Generate Single EML

生成：

`{{项目名称}}-邀标邮件.eml`

规则：

- 单一 EML；
- Bcc = 全部已确认供方邮箱；
- To 仅可填采购方明确提供的本方发送/归档邮箱，否则留空；
- Cc 仅可填明确确认的内部抄送邮箱；
- 正文只引用 02 最终需求清单和 03 招标意向征集登记表；
- 不引用 04 供方信息长名单；
- 不嵌入附件；
- 不自动发送。

### Step 6 — Prepare Attachment 1

附件1 = 人工确认的最终需求清单。

不得重新优化或改变 confirmed requirement。

### Step 7 — Calculate Bid Bond Variable（强制）

读取：

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

### Step 8 — Prepare Attachment 2 from Original Enterprise Template（强制）

附件2必须从仓库内企业原版模板：

`.agents/skills/sourcing-invitation/templates/招标意向征集登记表模板.xlsx`

按文件副本方式生成：

`copy original template → populate allowed current-project fields → replace N3 variable → save as {{项目名称}}-招标意向征集登记表.xlsx`

企业模板 `Sheet1!N3` 必须包含变量：

`{{投标保证金金额}}`

生成项目文件时，将其替换为 Step 7 的 `bid_bond_amount`。

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

### Step 9 — Integrity & Consistency Check（强制）

检查四个交付物：

#### 03 原模板及保证金

1. 输出从企业原模板复制，而非新建 workbook；
2. 原模板结构和样式未被无关修改；
3. `Sheet1!N3` 不再包含 `{{投标保证金金额}}`；
4. N3 金额 = `CEILING(采购清单预估总金额 × 1%, 1000)`；
5. 文件可正常打开且无公式错误。

#### 04 长名单

1. 长名单供方集合覆盖全部 `confirmed_initial_supplier_pool`；
2. 联系信息来源可追溯；
3. 缺失联系信息已留空并标记；
4. 无官方库外/未确认供方；
5. 所有 BCC 邮箱都能在长名单中找到对应记录。

#### 对外邮件保密

1. 供方邮箱只在 Bcc；
2. EML 不嵌入任何附件；
3. EML 正文仅引用 02、03；
4. 04 长名单不被引用、不被外发。

如果不能保证模板完整性：

`BLOCK: original intention form template integrity not preserved`

如果无法获得采购清单预估总金额：

`BLOCK: missing_procurement_estimated_total_amount`

如果 N3 金额计算或替换不一致：

`BLOCK: bid_bond_variable_mismatch`

如果 BCC 与长名单联系人邮箱不一致：

`BLOCK/HOLD: supplier_contact_bcc_mismatch`

### Step 10 — Output Package

固定输出：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
04 {{项目名称}}-供方信息长名单.xlsx
```

其中：

- 02、03 = 对外邮件附件；
- 04 = 采购内部文件，不外发。

并输出结构化 manifest。

### Step 11 — Human Send Checkpoint

生成完成不等于自动发送。采购员最终确认：

- Bcc；
- 邮件正文；
- 02、03 两个实际外发附件；
- 截止时间；
- 商务条款；
- 投标保证金金额；
- 04 长名单联系信息完整性。

## 5. Guardrails

- 官方供方库 ONLY；
- 不猜联系人、电话、邮箱；
- 供方邮箱仅进入 Bcc；
- 不自动发送；
- EML 不嵌入附件；
- 不改变 confirmed requirement；
- 不替供方填写意愿/能力；
- 不重建、重新设计或替换企业原版招标意向征集登记表模板；
- 不从历史项目模板继承未确认变量；
- 不把历史固定保证金金额作为当前项目保证金；
- 不从供应商报价空白列反推采购清单预估总金额；
- 保证金固定按当前规则 `1% + 千元向上取整` 计算，除非采购员明确修改当前项目规则；
- 04 长名单不得作为供方邮件附件，不得向任一供方披露其他供方联系信息。

## 6. Success Criteria

1. 仅生成一个 BCC EML；
2. 生成三个独立 Excel，其中 02、03 对外，04 仅内部使用；
3. 04 长名单覆盖全部人工确认初版供方；
4. 04 至少列出供方名称、联系人姓名、电话、邮箱，缺失字段明确标记且不猜测；
5. 每个 BCC 邮箱均可回溯到 04 长名单；
6. 03 登记表与企业原始模板结构/版式一致；
7. `Sheet1!N3` 的保证金金额来自当前项目预估总金额动态计算；
8. N3 金额满足 `CEILING(采购清单预估总金额 × 1%, 1000)`；
9. 供方填写区保持空白；
10. 停在人工发送确认节点。
