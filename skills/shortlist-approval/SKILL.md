---
name: shortlist-approval
description: 读取采购员已审核的供方短名单表，整理短名单决策过程，生成短名单报批邮件草稿，并形成供后续采购策略报告使用的结构化 Handoff。不得重新发明入围理由或替采购员作最终审批决定。
metadata:
  version: "0.6.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
  stage: "shortlist-approval"
---

# Shortlist Approval

## 1. Purpose

本 Skill 位于 `supplier-shortlist` 之后。

它不再重新评价供应商，而是将采购员已经审核/确认的短名单结果，整理为：

1. 短名单报批邮件草稿；
2. 短名单决策摘要；
3. `strategy_handoff`，供后续 `sourcing-strategy` 直接使用。

核心原则：

> Shortlist facts in, approval draft out.

## 2. Required Inputs

至少需要：

1. `supplier-shortlist` 输出的《{{项目名称}}供方短名单.xlsx》；
2. 采购员已经确认的入围/不入围结果，或表内明确的最终确认字段；
3. 当前项目名称和采购品类；
4. 当前采购阶段说明。

可选：

- 需求清单摘要；
- 历史采购分析 Handoff；
- 官方供方匹配 Handoff；
- 邀标/意向征集批次信息；
- 采购员希望写入邮件的补充说明；
- 审批收件人/抄送人（只有用户明确提供时才能写入）。

## 3. Do Not Re-score Suppliers

本 Skill 禁止重新进行：

- 复杂评分；
- 报价分析；
- 市场排名；
- 公开网络供应商比较；
- 自动改变短名单结论。

供应商的入围/未入围理由必须来自：

1. 已确认短名单表；
2. 供方真实回复；
3. 官方供方数据；
4. 采购员明确补充。

如果短名单仍是“建议入围 / 待澄清 / 不建议入围”的 AI 草稿，而采购员尚未确认：

- 可以生成“待审批/待确认”版邮件；
- 不得把“建议入围”改写成“已入围”。

## 4. Workflow

### Step 1 — Read Confirmed Shortlist

读取并区分：

- 入围；
- 不入围；
- 待澄清；
- 未回复/未完成确认。

保留每家供方的真实原因。

### Step 2 — Build Process Summary

统计可从当前资料明确得到的数量：

- 官方库长名单数量；
- 本轮沟通供方数量；
- 已回复供方数量；
- 建议/确认入围数量；
- 不入围数量；
- 待澄清数量。

没有数据则写 `未统计`，不得自行推算。

### Step 3 — Build Shortlist Summary

对入围供方只做简洁归纳：

```text
供方名称
入围状态
主要依据
待关注事项（如有）
```

不得添加表中不存在的“优势”。

### Step 4 — Build Exclusion Summary

不入围供方保留真实原因，例如：

- 明确无合作意向；
- 无法覆盖配送区域；
- 不接受账期；
- 不接受保证金；
- 关键资料未满足；
- 采购员确认的其他原因。

避免情绪化或评价性表达。

### Step 5 — Generate Approval Email Draft

邮件只需要解决四件事：

1. 项目是什么；
2. 本轮短名单征集/筛选情况；
3. 建议/确认的短名单有哪些；
4. 请审批什么。

默认使用 `templates/shortlist-approval-email.md`。

如果用户没有给审批人姓名：

- 使用 `各位领导/审批人` 或模板变量；
- 不得猜测姓名、职务或邮箱。

### Step 6 — Human Review

发送前必须由采购员确认：

- 项目名称；
- 入围供方名单；
- 未入围原因；
- 邮件统计数字；
- 审批动作；
- 收件人/抄送人。

本 Skill 只生成邮件草稿，不自动发送。

### Step 7 — Strategy Handoff

形成后续采购策略报告需要的信息：

```yaml
strategy_handoff:
  shortlist:
    longlist_count:
    contacted_count:
    replied_count:
    shortlisted_count:
    shortlisted_suppliers:
    excluded_suppliers:
    pending_suppliers:
  selection_rationale:
  supplier_market_observations:
  risk_flags:
  human_confirmation:
```

其中 `supplier_market_observations` 只能根据本轮官方库和真实回复概括，例如：

- 库内可响应供方数量有限；
- 多家供方对账期存在反馈；
- 部分供方区域覆盖不足。

不得扩展为未经调研的“整个市场行情”。

## 5. Output

主要交付物：

### A. 短名单报批邮件草稿

`{{项目名称}}-短名单报批邮件.md`

### B. Structured Handoff

`shortlist-approval-handoff.yaml`

不额外生成 PPT、复杂分析报告或评分表，除非用户另行要求。

## 6. Email Writing Rules

邮件风格：

- 简洁；
- 事实导向；
- 方便审批人快速判断；
- 不复制整张短名单表；
- 不堆砌所有供方回复细节。

推荐结构：

```text
主题
称呼

一、项目及本轮筛选情况
二、拟入围短名单
三、未入围/待澄清情况
四、报批事项

附件：
1. 供方短名单
2. 其他必要附件

落款
```

## 7. Guardrails

- 不改变采购员已经确认的短名单。
- 不把“建议”写成“批准”。
- 不自行新增供方。
- 不从公网补入供应商信息。
- 不虚构供方优势、合作业绩或不入围原因。
- 不猜审批人姓名、邮箱、职位。
- 不自动发送邮件。
- 所有统计数字必须可追溯到输入资料。

## 8. Success Criteria

1. 报批邮件中的供方名单与确认后的短名单表一致；
2. 入围/未入围原因不脱离真实资料；
3. 邮件能让审批人快速理解“为什么报、报哪些、需要批什么”；
4. 后续 `sourcing-strategy` 可以直接消费结构化短名单数据。
