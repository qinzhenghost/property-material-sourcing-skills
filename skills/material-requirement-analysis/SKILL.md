---
name: material-requirement-analysis
description: 分析物业物资类采购初版需求清单，识别物料级与项目级遗漏、歧义、隐藏分拆维度及冲突，生成最少必要澄清问题；P0 阻断项全部澄清后必须生成需求清单 Excel，并输出标准需求结构化结果与后续 sourcing-invitation Handoff。V1仅覆盖物业物资类采购与邀标制流程。
metadata:
  version: "0.1.2"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
---

# Material Requirement Analysis

## 1. Purpose

把采购方提供的初版物资需求转化为可用于后续官方供方库匹配和邀标的标准需求。

核心流程：

`初版需求 → 需求诊断 → P0/P1/P2 → 人工澄清 → P0清零 → 需求清单.xlsx → Structured Handoff → sourcing-invitation`

本 Skill 的核心不是替采购员补需求，而是：

1. 读取并结构化现有事实；
2. 识别缺失、歧义、冲突和隐藏分拆维度；
3. 只提出影响寻源或报价可比性的必要澄清；
4. 等待人工确认 P0；
5. **P0 全部解除后强制生成需求清单 Excel**；
6. 形成标准需求并交给 `sourcing-invitation`，由其先完成官方供方库匹配，再在人工确认供方后生成邀标材料。

## 2. Scope

- Domain: 物业物资类采购
- Procurement object: Goods / Materials only
- Sourcing method: 邀标制
- Downstream: `sourcing-invitation`
- V1 excludes: 保洁、秩序、绿化、维修等服务采购

## 3. Information States

所有信息必须区分：

- `FACT`：文件或用户明确确认的事实；
- `ASSUMPTION`：临时假设，不能自动升级为事实；
- `MISSING`：当前没有的信息；
- `HUMAN_DECISION`：必须由采购员决定。

## 4. Source Priority

1. 用户当前明确确认的澄清结果；
2. 当前项目最终/澄清版需求清单；
3. 当前项目初版需求清单；
4. `historical-procurement-analysis` 的 confirmed / calculated handoff；
5. 企业官方三级分类表；
6. 历史类似项目，仅用于提示，不得覆盖当前项目事实。

供方报价字段为空不等于采购需求缺失。

## 5. Document Role Detection

先判断输入属于：

- `initial_requirement`
- `clarified_requirement`
- `standard_requirement`
- `quotation_template`

典型供方响应字段：

- 含税单价
- 未税单价
- 税率
- 含税总价
- 报价单位
- 联系人
- 电话
- 供方备注

这些字段为空时不得作为需求缺口。

## 6. Workflow

### Step 1 — Parse Project Context

提取：

- 项目名称
- 采购周期
- 配送范围/交付地点
- 报价口径
- 预计量是否仅供参考
- 付款/账期
- 合作期限
- 评标/定标价格口径
- 拟中选供方数量
- 其他当前项目约束

缺失时标记，不自行补成事实。

### Step 2 — Parse Item Lines

至少提取：

- 原行号
- 区域/项目（如有）
- 物料名称
- 品牌
- 规格/型号/容量/包装
- 单位
- 起订量
- 年度预计采购量
- 技术/质量属性
- 备注

### Step 3 — Category Classification

优先映射企业官方：

`一级 → 二级 → 三级`

无法明确时只给候选并要求确认，不能伪装成官方分类。

### Step 4 — Universal Completeness Check

检查：

1. 名称能否唯一识别采购对象；
2. 单位是否明确；
3. 年度预计采购量是否明确；
4. 品牌/规格/型号/容量/材质/包装是否足以形成可比报价；
5. 品牌限定或等效替代规则是否明确；
6. 必要的质量标准/认证/样品/验收是否明确；
7. 配送区域是否明确；
8. 跨区域 SKU 是否需要按区域拆量；
9. 供应商能否仅凭当前信息准确报价。

### Step 5 — Hidden Dimension Detection

以下情况必须检查隐藏维度：

- 同一名称 + 同一单位重复出现；
- 同名行数量差异明显但无解释；
- 同名行可能对应不同城市、项目、品牌、规格、容量、包装、品质等级；
- 历史采购 Handoff 显示同 SKU 存在多区域，但需求清单未标区域。

候选隐藏维度包括：

- 区域/城市
- 项目/交付点
- 品牌
- 规格/容量/型号
- 包装规格
- 品质等级
- 技术参数

AI 不得自行猜测并改写需求。

### Step 6 — Project-Level Check

至少检查：

- 配送范围
- 报价口径/税运口径
- 年度预计量口径
- 付款/账期
- 合作周期
- 评标价格口径
- 拟中选供方数量

### Step 7 — Contradiction Check

检查：

- 标题与正文区域冲突；
- 单位与规格冲突；
- 总数量与区域拆分之和冲突；
- 含税/未税口径冲突；
- 不同版本数量/规格变化。

只报告差异，不自行判定哪个旧版本正确。

### Step 8 — Priority

#### P0 — Blocking

不澄清就无法合理寻源或报价不可比较，例如：

- 规格不足以识别物料；
- 区域未知且影响供货能力；
- 重复 SKU 无法判断差异；
- 年度预计采购量缺失且没有可用历史测算；
- 评标价格口径缺失。

#### P1 — Important

会明显影响价格、履约能力或合同争议，但不阻断生成澄清版需求清单。

#### P2 — Recommended

优化项，不阻断下一阶段。

### Step 9 — Clarification Questions

按 P0 → P1 → P2 输出。每个问题说明：

- 需要确认什么；
- 为什么重要；
- 不确认会影响什么。

### Step 10 — P0 Human Clarification Gate

若存在 P0：

- `requirement_status` 不得为 confirmed；
- 不进入 `sourcing-invitation`；
- 等待采购员回答。

采购员回答后重新检查 P0。

**一旦 P0 = 0，必须立即进入 Excel 交付阶段。**

P1/P2 尚未全部解决，不得成为“不生成 Excel”的理由。

### Step 11 — Mandatory Requirement Workbook Output

P0 全部解除后固定生成需求清单 Excel。

#### 11.1 原始输入为 Excel

优先：

1. 复制原需求清单为新文件；
2. 保留原有版式、标题、公式和企业模板结构；
3. 将采购员澄清确认内容写回对应行；
4. 新增字段时尽量在原表结构内合理增加；
5. 不修改原始文件本身。

#### 11.2 无可复用 Excel 模板

使用：

`templates/标准需求清单模板.xlsx`

#### 11.3 文件命名

P0 已清零但仍有 P1/P2：

`{{项目名称}}-澄清版需求清单.xlsx`

关键需求与项目商务条件均已确认：

`{{项目名称}}-最终需求清单.xlsx`

#### 11.4 Excel 至少包含

- 序号
- 区域（多区域时必须显式存在）
- 名称
- 品牌（如有要求）
- 规格
- 单位
- 起订量（如有）
- 年度预计采购量
- 备注

项目级信息应保留：

- 项目名称
- 采购周期
- 配送范围
- 报价口径
- 付款/账期
- 合作期限
- 预计量约束说明

#### 11.5 供方报价字段

原模板若包含以下列，应保留但保持空白：

- 含税单价
- 未税单价
- 税率
- 含税总价

不得把历史参考价格、AI测算价或旧报价写入这些供方待填列。

#### 11.6 P1/P2 留痕

仍未解决的 P1/P2：

- 不得隐藏；
- 在 Excel 备注或单独“待确认事项”区域留痕；
- 不能改变已确认采购事实。

详细规则见：`references/requirement-workbook-output-rules.md`。

### Step 12 — Build Standard Requirement

P0 清零后生成符合 `schemas/requirement.schema.yaml` 的标准对象，并记录 Excel 输出状态。

不得改变采购员确认的：

- 数量
- 区域
- 品牌
- 规格
- 商务规则

### Step 13 — Structured Handoff

P0 清零后必须同时输出：

```yaml
handoff:
  workflow_stage: requirement_clarified
  procurement_domain: property_material
  sourcing_method: invitation_tender
  requirement_status: clarified_or_confirmed
  p0_blocker_count: 0
  requirement_workbook:
    generated: true
    filename: "{{项目名称}}-澄清版需求清单.xlsx"
    source_mode: copied_from_input_or_standard_template
  category_path: {}
  project_constraints: {}
  item_count: 0
  unresolved_p1: []
  unresolved_p2: []
  assumptions: []
  human_decision:
    p0_confirmed: true
  next_skill: sourcing-invitation
  next_phase: official_supplier_matching
```

如果 `requirement_workbook.generated != true`，则该 Skill 不得宣称本阶段交付完成。

## 7. Output Contract

### P0 尚未清零

输出：

1. 需求诊断摘要；
2. 已确认事实；
3. P0/P1/P2；
4. 澄清问题；
5. 暂不进入下游。

此时 Excel 可以不生成。

### P0 已清零

必须同时交付：

1. 需求诊断/澄清摘要；
2. `{{项目名称}}-澄清版需求清单.xlsx` 或 `{{项目名称}}-最终需求清单.xlsx`；
3. 标准 Requirement 结构化结果；
4. Structured Handoff；
5. 未解决 P1/P2（如有）。

缺少 Excel 即视为 Skill 执行未完成。

## 8. Golden Case Guardrails

1. 空白供方报价列不能被判定为需求缺失；
2. 同名同单位重复行必须触发隐藏维度检查；
3. 历史采购显示多区域时，不得依赖行序猜区域；
4. 人工澄清后的规格/区域必须进入 Excel；
5. P0 清零后必须生成 Excel；
6. P1/P2 不得阻断 Excel 生成；
7. 供方报价字段保持空白；
8. Excel 与 Structured Handoff 的 SKU、区域、规格和数量一致。

## 9. Human Review

采购员最终确认：

- P0 澄清结果；
- 区域/规格/品牌映射；
- 年度预计采购量；
- 项目级商务条件；
- 最终需求清单。

## 10. Success Criteria

1. P0 能准确识别并通过最少问题澄清；
2. P0 清零后必有 Excel 交付物；
3. Excel 保留原模板风格或使用标准模板；
4. P1/P2 未解决时能留痕而非阻断；
5. 供方报价字段不被预填；
6. Excel、Requirement Schema、Handoff 三者一致；
7. `sourcing-invitation` 可直接消费需求清单和 Handoff，并从其 Phase A 开始官方供方匹配。
