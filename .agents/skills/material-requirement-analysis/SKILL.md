---
name: material-requirement-analysis
description: 分析物业物资类采购初版需求清单，识别物料级与项目级遗漏、歧义、隐藏分拆维度及冲突，生成最少必要澄清问题，并在人工确认后输出标准需求结构化结果。V1仅覆盖物业物资类采购与邀标制流程。
metadata:
  version: "0.1.0"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
---

# Material Requirement Analysis

## 1. Purpose

把采购方提供的初版物资需求清单转化为可用于后续官方供方库寻源、意向征集、邀标和回标比价的标准需求。

本 Skill 的价值不是“替采购员补需求”，而是：

1. 读取并结构化现有事实；
2. 识别缺失、歧义、矛盾和隐藏维度；
3. 解释这些问题为什么会影响寻源、报价可比性或后续评标；
4. 只提出阻碍下一阶段或显著影响报价可比性的澄清问题；
5. 等待人工回答；
6. 根据人工确认结果生成标准需求与结构化 handoff。

## 2. Parent / Scope

- Domain: 物业物资类采购
- Procurement object: Goods / Materials only
- Sourcing method: 邀标制
- V1 excludes: 保洁、秩序、绿化、维修等服务类采购
- Downstream target: `official-supplier-matching`

## 3. Core Principle

> Do not source against an unclear requirement.

但“需求不完整”不等于必须无休止追问。

必须把信息分成：

- **FACT**：文件或用户明确确认的事实；
- **ASSUMPTION**：为了分析而暂时采用的假设，必须显式标记；
- **MISSING**：当前没有的信息；
- **HUMAN_DECISION**：必须由采购员确认的事项。

永远不得把 ASSUMPTION 自动升级成 FACT。

## 4. Authoritative Data Sources

按优先级使用：

1. 当前用户明确确认的需求或澄清结果；
2. 当前项目的最终确认版需求清单；
3. 当前项目的初版需求清单；
4. 企业官方《物资类供应商三级分类表》——仅用于品类分类与属性参考；
5. 历史类似项目——仅用于提示遗漏，不得覆盖当前项目要求。

### Source-of-Truth Rules

- 企业官方三级分类是品类命名的优先来源。
- 品牌、型号、规格、质量标准、数量、区域、账期、评标规则等不得由 AI 凭经验自动补成采购要求。
- 历史项目只能作为“建议澄清”的证据，不是当前项目事实。
- 供方报价字段为空，不得直接判定为采购需求遗漏。

## 5. Required Inputs

至少需要一种：

- Excel / CSV 物料需求清单；
- Word / Markdown / 文本形式需求；
- 用户直接粘贴的物料列表。

可选：

- 企业官方物资三级分类表；
- 同品类历史最终需求清单；
- 当前项目背景说明。

如果没有官方三级分类，允许进行“临时品类推断”，但必须标记为 `ASSUMPTION`，不得伪装成官方分类。

## 6. Document Role Detection

读取文件后先判断它属于：

- `initial_requirement`
- `clarified_requirement`
- `standard_requirement`
- `quotation_template`

不得看到“含税单价/未税单价/税率/含税总价”等空白列就认为需求缺失。

### Supplier response fields examples

以下字段通常属于供方响应区：

- 报价单位
- 联系人
- 电话
- 地址
- 报价时间
- 含税单价
- 未税单价
- 税率
- 含税总价
- 供方备注

如果上下文表明它们是供方待填写字段，将其写入 `supplier_response_fields`，不进入需求缺口计数。

## 7. Workflow

### Step 1 — Parse Project Context

提取：

- 项目名称/框架名称
- 采购周期
- 配送范围/交付地点
- 报价口径（综合价、含税、含运等）
- 预计量是否仅供参考
- 下单渠道/采购平台要求
- 付款与账期
- 合作期限
- 评标/定标规则
- 计划中选供方数量
- 招标对接信息

没有就标记 MISSING，不自动补。

### Step 2 — Parse Item Lines

至少提取：

- 行号
- 物料名称
- 品牌（如有）
- 规格/型号（如有）
- 计量单位
- 起订量（如有）
- 预计采购量
- 区域数量拆分（如有）
- 技术/质量属性（如有）
- 备注

### Step 3 — Classify Material Category

优先把每个 SKU 映射到企业官方：

`一级 → 二级 → 三级`

若无法明确映射：

- 不强制猜测；
- 给出最多 1–3 个候选分类；
- 标记需要人工确认。

### Step 4 — Universal Completeness Check

对所有物资检查：

1. 物料名称是否能唯一识别采购对象；
2. 计量单位是否明确；
3. 预计数量是否明确；
4. 规格/型号/尺寸/容量/材质/包装等是否足以形成可比报价；
5. 是否存在品牌限定或等效替代规则；
6. 是否需要质量标准、认证、样品、质保或验收条件；
7. 交付区域是否明确；
8. 若同 SKU 跨区域，是否需要数量按区域拆分；
9. 供应商是否能仅凭当前信息准确报价。

### Step 5 — Category-Specific Attribute Check

根据品类动态检查关键属性。

示例：

- 粮油：品牌、净含量/容量、等级、加工/工艺要求、是否非转基因、包装单位；
- 清洁用品：材质、尺寸、容量、浓度/成分、包装规格、适配设备；
- 消防用品：品牌/型号、规格参数、认证要求、压力/容量、有效期、执行标准；
- 工程耗材：材质、型号、尺寸、电气/机械参数、接口、适配对象、执行标准。

**禁止把示例字段机械套到所有品类。**

### Step 6 — Hidden Dimension Detection

这是 V1 的重点规则。

当出现以下任一情况时，必须检查是否存在未表达的隐藏分拆维度：

- 同一物料名称 + 同一单位重复出现；
- 同名行存在不同参考价；
- 同名行数量差异很大但没有区域/项目/规格说明；
- 同名行在后续文件中出现不同城市或不同交付点；
- 同一名称实际可能对应不同品牌、容量、型号、包装或等级。

候选隐藏维度包括但不限于：

- 品牌
- 规格/容量/型号
- 城市/区域
- 项目/交付点
- 包装规格
- 品质等级
- 技术参数

不得自行选择其中一个维度并改写需求；应提出澄清问题。

### Step 7 — Project-Level Completeness Check

在进入官方供方库匹配前，至少检查：

- 配送/交付范围
- 报价口径
- 税/运费口径
- 采购数量口径（预计量是否绑定）
- 付款/账期
- 合作/合同周期
- 评标依据
- 拟中选供方数量

以下信息若缺失且会改变供方适配性或报价可比性，应列为 blocker。

### Step 8 — Contradiction & Formula Check

检查：

- 标题与正文区域是否冲突；
- 单位与规格是否冲突；
- 预计量与区域拆分之和是否一致；
- 含税/未税口径是否混用；
- 总价公式是否与说明一致；
- 同一项目不同版本数量是否发生变化。

版本发生变化时只报告差异，不擅自判断哪个数值“正确”。

### Step 9 — Prioritize Findings

把发现分为：

#### P0 — Blocking

不澄清就无法合理寻源或报价不可比较。

例如：

- 规格不足以唯一识别物料；
- 交付区域未知且影响供货能力；
- 同名重复行无法判断差异；
- 评标价格口径未知。

#### P1 — Important

会显著影响价格、供方能力或后续争议。

#### P2 — Recommended

有助于提高质量，但不阻碍进入下一步。

### Step 10 — Generate Clarification Questions

问题必须：

- 一次只问一个决策点；
- 尽量给出结构化选项；
- 说明“为什么需要确认”；
- 说明“不确认会影响什么”；
- 优先 P0，再 P1；
- 不重复询问文件中已明确的信息。

推荐格式：

| 优先级 | 对象 | 缺口/歧义 | 澄清问题 | 为什么重要 | 影响 |
|---|---|---|---|---|---|

### Step 11 — Wait for Human Clarification

若存在 P0：

- 不得把需求标记为 confirmed；
- 不得建议开始官方供方库匹配；
- 等待采购员回答。

如果用户明确要求“先继续”，可以带着 ASSUMPTION 输出临时版本，但 `requirement_status` 必须保持 `pending` 或 `partially_clarified`。

### Step 12 — Build Standard Requirement

人工确认后：

- 生成符合 `schemas/requirement.schema.yaml` 的标准对象；
- 保留原字段与新增字段的差异记录；
- 不改变采购员确认的数量、规格、品牌或商务规则；
- 将报价字段与需求字段分离。

### Step 13 — Structured Handoff

只有人工确认且不存在 P0 blocker 时，才能输出：

```yaml
handoff:
  workflow_stage: requirement_confirmed
  procurement_domain: property_material
  sourcing_method: invitation_tender
  requirement_status: confirmed
  category_path:
    level_1: 物资类
    level_2: null
    level_3: null
  project_constraints:
    delivery_regions: []
    payment_terms: null
    evaluation_rule: null
  item_count: 0
  unresolved_items: []
  assumptions: []
  human_decision:
    confirmed: true
  next_skill: official-supplier-matching
```

## 8. Output Format

默认按以下顺序输出：

### A. 需求诊断摘要

- 清单行数
- 可识别 SKU 数
- P0/P1/P2 数量
- 当前是否可以进入供方寻源

### B. 已确认事实

只列 FACT。

### C. 需求缺口与歧义

使用表格。

### D. 澄清问题

按 P0 → P1 → P2 排序。

### E. 澄清后的标准需求

仅在用户回答后生成。

### F. Structured Handoff

仅在满足进入下一 Skill 的条件时生成。

## 9. Golden-Case Rules Learned from Current Samples

从“米面粮油物资框架”样本必须通过以下测试：

1. 初版 18 行物料不能因为“含税单价/含税总价为空”被判定为需求缺失；它们是供方报价字段。
2. 初版物料缺少品牌、规格，必须识别为关键缺口。
3. “特香低芥酸菜籽油（非转基因）”同名同单位出现 4 次，必须触发隐藏维度检查。
4. 其中不同 C 端参考价提示可能存在不同容量/规格，但 AI 不得直接断言规格。
5. 最终版补充了品牌与规格，例如 5L/桶、4L/桶、1.8L/瓶、5KG/袋。
6. 最终版新增综合报价、含税含运、120 天账期、重庆/贵州配送范围、1 年合作期、未税总价最低等项目级条件。
7. 即使“最终版”中同 SKU 仍重复，后续价格对比表出现重庆/贵州城市拆分，因此必须提示检查行级区域拆分是否需要进入标准需求。
8. 后续文件的数量发生变化时，只报告版本差异并要求确认，不自动覆盖当前需求数量。

## 10. Human Review

必须由采购员确认：

- 品牌限制；
- 规格/型号；
- 数量及区域拆分；
- 商务条件；
- 评标规则；
- 是否允许替代品；
- 是否可以进入官方供方库寻源。

## 11. Guardrails

- 不凭空补需求。
- 不把市场常见规格写成当前项目规格。
- 不把历史项目字段当成当前项目事实。
- 不把供方待填写字段当需求缺口。
- 不在需求未确认时生成最终邀标供方名单。
- 不执行公网供方搜索；供方来源由后续 `official-supplier-matching` 严格限制为内部官方库。
- 不把“最终版”文件视为永远正确；仍需检查内部一致性与隐藏维度。

## 12. Review Checklist

完成前逐项检查：

- [ ] 是否正确识别文档角色？
- [ ] 是否分离采购需求字段和供方响应字段？
- [ ] 是否提取项目级商务条件？
- [ ] 是否逐 SKU 检查规格完整性？
- [ ] 是否检查同名重复行的隐藏维度？
- [ ] 是否检查区域数量拆分？
- [ ] 是否区分 FACT / ASSUMPTION / MISSING / HUMAN_DECISION？
- [ ] 是否只提出必要问题？
- [ ] 是否避免把 AI 建议写成采购要求？
- [ ] 存在 P0 时是否阻止进入下一 Skill？
- [ ] 是否在人工确认后才标记 `requirement_status: confirmed`？

## 13. Success Criteria

本 Skill 成功的标准不是“输出一份看起来完整的清单”，而是：

1. 采购员能快速看懂原始需求哪里不能直接寻源；
2. 澄清问题少而关键；
3. 澄清后不同供方能基于同一口径报价；
4. 下游官方供方库匹配能获得明确的品类、区域和关键规格输入；
5. 所有 AI 推断与采购员确认事实可追溯区分。
