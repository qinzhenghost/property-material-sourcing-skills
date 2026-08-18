---
name: supplier-shortlist
description: 读取 sourcing-invitation 已确认的供方信息长名单及供方真实回复，汇总形成供方短名单筛选表；采购员人工确认最终短名单后，在同一 Skill 内继续生成短名单报批邮件草稿与 sourcing-strategy 所需的 strategy_handoff。V1不做复杂评分、报价排名或自动定标。
metadata:
  version: "0.5.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  stage: "supplier-shortlist-and-approval"
---

# Supplier Shortlist

## 1. Purpose

本 Skill 合并原 `supplier-shortlist` 与 `shortlist-approval`，覆盖一个完整业务阶段：

`供方回复 → 短名单筛选表 → 人工确认最终短名单 → 报批邮件草稿 → strategy_handoff`

分为两个强制阶段：

- **Phase A — Supplier Reply & Shortlist Draft**：汇总供方真实回复，形成短名单筛选表草稿。
- **Phase B — Shortlist Approval Package**：只有采购员明确确认最终短名单后，才生成正式报批材料和后续策略 Handoff。

人工确认节点保留在 Skill 内部，不再单独调用 `shortlist-approval`。

## 2. Scope / Upstream / Downstream

- Domain: 物业物资类采购
- Sourcing method: 邀标制
- Required upstream: `sourcing-invitation`
- Default downstream: `sourcing-strategy`

本 Skill 不负责：

- 复杂评分；
- 报价排名；
- 成本模型；
- 自动定标；
- 从公网新增供方；
- 替采购员作最终短名单决定；
- 在 Phase B 重新评价或重新排序采购员已确认的短名单。

## 3. Required Inputs

至少需要：

1. `sourcing-invitation` 输出的 `{{项目名称}}-供方信息长名单.xlsx` 或等价 confirmed initial supplier pool；
2. 供方邮件回复和/或回传的《招标意向征集登记表》；
3. 当前项目已确认的关键邀标条件；
4. `templates/供方短名单模板.xlsx`。

Phase B 额外必须具备：

5. 采购员对最终入围 / 不入围 / 待澄清结果的明确确认。

可选：

- 官方供方库中的注册资本、供方库分类、合作历史、联系人、邮箱、电话；
- 采购员人工补充的已核实合作业绩、仓储/运输/授权/资质信息；
- 审批收件人 / 抄送人（仅在用户明确提供时使用）。

## 4. Source of Truth

### 供应商身份

必须来自 `sourcing-invitation` 已确认初版供方池 / 供方信息长名单。

不得新增不在该集合中的公网供方。

### 官方库字段优先用于

- 是否有合作历史；
- 注册资本；
- 供方库分类；
- 联系人、邮箱、电话；
- 其他官方明确字段。

### 供方回复优先用于

- 是否有合作意向；
- 是否可供应全部清单；
- 配送区域；
- 账期接受情况；
- 保证金接受情况；
- 货期；
- 仓储、车辆、团队；
- 品牌授权、代理情况；
- 合作案例；
- 明确不参与原因。

### Human Decision

最终短名单状态只能由采购员确认。

AI 的 `建议入围 / 待澄清 / 不建议入围` 不等于最终结果。

## 5. Phase A — Supplier Reply & Shortlist Draft

### Step A1 — Load Confirmed Longlist

以 `sourcing-invitation` 人工确认后的供方信息长名单为底表。

要求：

- 保留全部本轮已确认邀标供方；
- 不因未回复而删除供方；
- 不新增长名单外供方。

### Step A2 — Merge Official Registry Data

尽可能补入：

- 合作历史；
- 注册资本；
- 供方库分类；
- 邮箱；
- 电话。

无官方字段则留空/待补充，不推断。

### Step A3 — Parse Supplier Replies

逐家提取与当前品类相关的真实回复：

- 头部企业合作案例；
- 工厂 / 代理 / 品牌授权 / 资质；
- 仓储面积及库存能力；
- 配送车辆及团队；
- 配送区域；
- 是否供应全部清单；
- 是否接受账期；
- 是否接受保证金；
- 是否有合作意向；
- 明确不参与原因。

禁止：

- 把“未回复”写成“否”；
- AI 根据公司名称猜测能力；
- 修改供方真实回复含义。

缺失内容使用 `未反馈` / `待澄清`；明确无需继续收集时可使用 `/`。

### Step A4 — Fill Enterprise Shortlist Template

必须使用：

`.agents/skills/supplier-shortlist/templates/供方短名单模板.xlsx`

保持企业原版式，至少保留以下列：

| 列 | 字段 |
|---|---|
| A | 序号 |
| B | 库内长名单 |
| C | 头部企业合作业绩（2家及以上） |
| D | 工厂、代理商、资质证书、基地、设施设备、仓储空间、运输工具等 |
| E | 公司性质（是否为个体户） |
| F | 是否有合作意向 |
| G | 是否有合作历史 |
| H | 注册资本（万） |
| I | 供方库分类 |
| J | 是否入围及未入围原因 |
| K | 邮箱 |
| L | 电话 |

不得自行增加复杂评分列。

### Step A5 — AI Recommendation in Column J

Phase A 只允许三类：

#### `建议入围`

格式：`建议入围：{可追溯的主要满足条件}`

#### `待澄清`

格式：`待澄清：{具体缺失或冲突项}`

#### `不建议入围`

只在真实资料明确支持时使用，例如：

- 明确不参加；
- 无法覆盖配送区域；
- 明确不接受账期；
- 明确不接受保证金；
- 明确不能满足当前项目关键条件。

不得用“无合作历史”或“注册资本较低”等关注条件单独判定不入围，除非当前企业规则明确将其设为必要门槛。

### Step A6 — Generate Shortlist Workbook

生成：

`{{项目名称}}-供方短名单.xlsx`

此时状态必须是：

`shortlist_status = draft_pending_human_confirmation`

### Step A7 — Human Shortlist Checkpoint（强制）

输出短名单草稿后必须暂停，等待采购员逐项或整体确认：

- shortlisted；
- excluded；
- pending_clarification。

未确认前：

- 不得把“建议入围”写成“已入围”；
- 不得生成可报批状态的 Handoff；
- 默认不进入 sourcing-strategy。

如果采购员明确要求，可提前生成带 `DRAFT / 待确认` 标识的报批邮件草稿，但不能写成已确认短名单。

## 6. Phase B — Shortlist Approval Package

只有 `human_shortlist_checkpoint.status = confirmed` 才进入。

### Step B1 — Freeze Human-Confirmed Decisions

读取并锁定：

- 最终入围；
- 最终不入围；
- 待澄清；
- 未回复/未完成确认。

Phase B **不得重新评分、重新排序或改变这些结论**。

### Step B2 — Finalize Shortlist Workbook

同一企业短名单模板中，将人工确认结果反映到最终版本。

输出仍为：

`{{项目名称}}-供方短名单.xlsx`

但结构化状态改为：

`shortlist_status = human_confirmed`

必须保留不入围供方及真实原因，便于过程留痕。

### Step B3 — Build Process Summary

统计可追溯的：

- 初版邀标长名单数量；
- 本轮实际沟通数量；
- 已回复数量；
- 最终入围数量；
- 不入围数量；
- 待澄清数量。

无可靠数据写 `未统计`，不得自行推算。

### Step B4 — Generate Approval Email Draft

使用：

`.agents/skills/supplier-shortlist/templates/shortlist-approval-email.md`

生成：

`{{项目名称}}-短名单报批邮件.md`

邮件只解决：

1. 项目是什么；
2. 本轮征集/筛选情况；
3. 最终确认短名单有哪些；
4. 不入围 / 待澄清概况；
5. 请审批什么。

如果审批人姓名、邮箱、职位未明确：

- 保留模板变量或使用泛称；
- 不猜测。

邮件只是草稿，不自动发送。

### Step B5 — Build Strategy Handoff

生成：

`{{项目名称}}-shortlist-handoff.yaml`

至少包含：

```yaml
strategy_handoff:
  workflow_stage: shortlist_confirmed
  shortlist:
    longlist_count:
    contacted_count:
    replied_count:
    shortlisted_count:
    excluded_count:
    pending_count:
    shortlisted_suppliers: []
    excluded_suppliers: []
    pending_suppliers: []
  selection_rationale: []
  supplier_market_observations: []
  risk_flags: []
  human_confirmation:
    shortlist_confirmed: true
  next_skill: sourcing-strategy
```

`supplier_market_observations` 只能概括本轮官方库和真实回复，例如“可响应供方数量有限”“多家供方对账期存在反馈”“部分供方区域覆盖不足”，不得扩展成未经调研的整个市场行情。

### Step B6 — Human Approval Draft Review

采购员在发送报批邮件前确认：

- 最终短名单；
- 未入围原因；
- 邮件统计数字；
- 报批事项；
- 收件人 / 抄送人；
- 短名单附件。

## 7. Output Contract

### Phase A 未完成人工确认

主要交付：

1. `{{项目名称}}-供方短名单.xlsx`（draft_pending_human_confirmation）；
2. 短名单建议摘要；
3. 待澄清事项；
4. Human Checkpoint。

### Phase B 人工确认后

固定交付：

1. `{{项目名称}}-供方短名单.xlsx`（human_confirmed）；
2. `{{项目名称}}-短名单报批邮件.md`；
3. `{{项目名称}}-shortlist-handoff.yaml`。

下游：`sourcing-strategy`。

## 8. Guardrails

- 官方确认长名单是供应商身份 Source of Truth；
- 供方真实回复是参与意愿与条件 Source of Truth；
- 不新增公网供方；
- 不虚构业绩、资质、仓储、车辆、授权或合作意愿；
- 不把未回复自动判定为否；
- 不用非必要关注条件替代必要条件；
- AI建议不等于人工确认；
- Phase B 不重新评价已确认短名单；
- 不猜审批人姓名、邮箱或职位；
- 不自动发送邮件；
- 所有统计数字和理由必须可追溯。

## 9. Success Criteria

1. 本轮邀标供方均被正确汇总；
2. 供方回复准确映射到企业短名单模板；
3. AI短名单建议与人工最终决定严格分离；
4. Phase A 必须停在人工确认节点；
5. Phase B 不改变采购员确认结果；
6. 报批邮件与最终短名单一致；
7. `strategy_handoff` 可被 `sourcing-strategy` 直接消费。
