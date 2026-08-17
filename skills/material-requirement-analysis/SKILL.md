---
name: material-requirement-analysis
description: 分析物业物资类采购初版需求清单，识别物料级与项目级遗漏、歧义、隐藏分拆维度及冲突，生成最少必要澄清问题；P0 阻断项全部澄清后必须生成需求清单 Excel，并输出标准需求结构化结果与后续官方供方匹配 Handoff。V1仅覆盖物业物资类采购与邀标制流程。
metadata:
  version: "0.1.1"
  domain: "property-material-procurement"
  sourcing_method: "invitation-tender"
---

# Material Requirement Analysis

## 1. Purpose

把采购方提供的初版物资需求转化为可用于后续官方供方库匹配和邀标的标准需求。

核心流程：

`初版需求 → 需求诊断 → P0/P1/P2 → 人工澄清 → P0清零 → 需求清单.xlsx → Structured Handoff`

本 Skill 的核心不是替采购员补需求，而是：

1. 读取并结构化现有事实；
2. 识别缺失、歧义、冲突和隐藏分拆维度；
3. 只提出影响寻源或报价可比性的必要澄清；
4. 等待人工确认 P0；
5. **P0 全部解除后强制生成需求清单 Excel**；
6. 形成标准需求和 `official-supplier-matching` Handoff。

## 2. Scope

- Domain: 物业物资类采购
- Procurement object: Goods / Materials only
- Sourcing method: 邀标制
- Downstream: `official-supplier-matching`
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

典型供方响应字段：含税单价、未税单价、税率、含税总价、报价单位、联系人、电话、供方备注。这些字段为空时不得作为需求缺口。

## 6. Workflow

### Step 1 — Parse Project Context

提取项目名称、采购周期、配送范围、报价口径、预计量约束、付款/账期、合作期限、评标/定标价格口径、拟中选供方数量等。缺失时标记，不自行补成事实。

### Step 2 — Parse Item Lines

至少提取原行号、区域/项目、物料名称、品牌、规格/型号/容量/包装、单位、起订量、年度预计采购量、技术/质量属性、备注。

### Step 3 — Category Classification

优先映射企业官方 `一级 → 二级 → 三级`。无法明确时只给候选并要求确认。

### Step 4 — Universal Completeness Check

检查名称、单位、年度预计采购量、品牌/规格/容量/材质/包装、替代规则、质量/认证/验收、配送区域、跨区域拆量，以及当前信息是否足够供方准确报价。

### Step 5 — Hidden Dimension Detection

同名同单位重复行、数量差异明显、多区域历史数据、可能存在不同品牌/规格/容量/包装/品质等级时必须触发隐藏维度检查。AI 不得自行猜测并改写需求。

### Step 6 — Project-Level Check

至少检查配送范围、报价税运口径、年度预计量口径、付款/账期、合作周期、评标价格口径、拟中选供方数量。

### Step 7 — Contradiction Check

检查区域、单位与规格、总量与区域拆分、税口径、不同版本数量/规格差异。只报告差异。

### Step 8 — Priority

#### P0 — Blocking

不澄清就无法合理寻源或报价不可比较，例如规格不足、区域未知、重复 SKU 无法区分、年度预计采购量缺失且无历史测算结果、评标价格口径未知。

#### P1 — Important

会明显影响价格、履约或合同争议，但不阻断生成澄清版需求清单。

#### P2 — Recommended

优化项，不阻断下一阶段。

### Step 9 — Clarification Questions

按 P0 → P1 → P2 输出，并说明确认点、原因和影响。

### Step 10 — P0 Human Clarification Gate

存在 P0 时，不得进入官方供方匹配。采购员回答后重新检查。

**一旦 P0 = 0，必须立即进入 Excel 交付阶段。P1/P2 尚未全部解决，不得成为“不生成 Excel”的理由。**

### Step 11 — Mandatory Requirement Workbook Output

P0 全部解除后固定生成需求清单 Excel。

#### 原始输入为 Excel

优先复制原需求清单为新文件，保留原版式、标题、公式和企业模板结构，将澄清结果写回对应行；不修改原始文件。

#### 无可复用 Excel

使用 `templates/标准需求清单模板.xlsx`。

#### 文件命名

- P0 已清零但仍有 P1/P2：`{{项目名称}}-澄清版需求清单.xlsx`
- 关键需求及项目商务条件均确认：`{{项目名称}}-最终需求清单.xlsx`

#### Excel 必须包含

至少：序号、区域（多区域时显式存在）、名称、品牌（如有要求）、规格、单位、起订量（如有）、年度预计采购量、备注。

项目级信息应保留项目名称、采购周期、配送范围、报价口径、付款/账期、合作期限和预计量约束说明。

#### 供方报价字段

原模板包含含税单价、未税单价、税率、含税总价时保留但保持空白。不得将历史参考价格、AI测算价或旧报价写入。

#### P1/P2 留痕

未解决的 P1/P2 在 Excel 备注或“待确认事项”区域留痕，不得隐藏。

详细规则见 `references/requirement-workbook-output-rules.md`。

### Step 12 — Build Standard Requirement

P0 清零后生成符合 `schemas/requirement.schema.yaml` 的标准对象，并记录 Excel 输出状态。不得改变采购员确认的数量、区域、品牌、规格和商务规则。

### Step 13 — Structured Handoff

P0 清零后必须同时输出 Excel 和 Handoff。如果 `requirement_workbook.generated != true`，不得宣称本阶段交付完成。

## 7. Output Contract

### P0 尚未清零

输出诊断摘要、事实、P0/P1/P2、澄清问题；Excel 可以不生成。

### P0 已清零

**必须同时交付：**

1. 需求诊断/澄清摘要；
2. `{{项目名称}}-澄清版需求清单.xlsx` 或 `{{项目名称}}-最终需求清单.xlsx`；
3. 标准 Requirement 结构化结果；
4. Structured Handoff；
5. 未解决 P1/P2（如有）。

缺少第 2 项 Excel 即视为 Skill 执行未完成。

## 8. Golden Case Guardrails

1. 报价字段为空不能判定为需求缺失；
2. 同名同单位重复行必须检查隐藏维度；
3. 多区域重复 SKU 不得依赖行序猜区域；
4. 人工澄清后的规格/区域必须进入 Excel；
5. P0 清零后必须生成 Excel；
6. P1/P2 不阻断 Excel；
7. 供方报价字段保持空白；
8. Excel 与 Structured Handoff 的 SKU、区域、规格和数量一致。

## 9. Human Review

采购员最终确认 P0 澄清结果、区域/规格/品牌映射、年度预计采购量、项目级商务条件和最终需求清单。

## 10. Success Criteria

1. P0 通过最少问题澄清；
2. P0 清零后必有 Excel；
3. Excel 保留原模板风格或使用标准模板；
4. P1/P2 可留痕而不阻断；
5. 供方报价字段不被预填；
6. Excel、Requirement、Handoff 三者一致；
7. `official-supplier-matching` 可直接消费需求清单和 Handoff。
