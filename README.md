# Property Material Sourcing Skills — v0.7.5

物业物资类 AI 邀标寻源 Skill 库。

## Codex 运行入口

```text
.agents/
└── skills/
    ├── historical-procurement-analysis/
    ├── material-requirement-analysis/
    ├── official-supplier-matching/
    ├── sourcing-invitation/
    ├── supplier-shortlist/
    ├── shortlist-approval/
    └── sourcing-strategy/
```

使用 Codex 打开仓库后，可通过 `/skills` 或 `$skill-name` 显式测试。

> `skills/` 暂保留历史兼容；测试和后续开发以 `.agents/skills/` 为主。

## 当前流程

```text
历史采购数据（可选）
        ↓
historical-procurement-analysis
        ├─ 用户选择协议编号
        ├─ 仅保留 已完成 / 执行中
        ├─ 不足12个月 → 折算12个月年度基线
        ├─ 确认年度增量比例
        └─ 输出历史测算需求清单.xlsx
        ↓
material-requirement-analysis
        ├─ 需求诊断
        ├─ P0 / P1 / P2 澄清
        ├─ P0清零
        ├─ 强制输出需求清单.xlsx
        └─ Structured Handoff
        ↓
official-supplier-matching
        ↓
sourcing-invitation
        ↓
supplier-shortlist
        ↓
shortlist-approval
        ↓
sourcing-strategy
        ↓
采购方案报告.docx
```

## 当前 Skill

| Skill | 作用 | 状态 |
|---|---|---|
| `historical-procurement-analysis` | 协议选择、有效订单过滤、12个月年化、年度增量、历史测算需求清单 | ✅ |
| `material-requirement-analysis` | 需求诊断、P0澄清、强制生成澄清/最终需求清单 Excel | ✅ |
| `official-supplier-matching` | 仅从官方供方库匹配候选供方 | ✅ |
| `sourcing-invitation` | 生成邀标三件套 | ✅ |
| `supplier-shortlist` | 根据供方回复生成短名单表 | ✅ |
| `shortlist-approval` | 短名单报批邮件及策略 Handoff | ✅ |
| `sourcing-strategy` | 生成采购方案/策略报告 | ✅ |

## historical-procurement-analysis v0.5.5

历史分析必须依次经过：

1. 用户明确选择一个或多个协议编号；
2. 仅保留 `已完成 / 执行中` 订单；
3. 不足12个月的数据按实际覆盖周期折算为12个月年度基线；
4. 有两年度可比数据时计算年度增量；
5. 无两年度数据时由用户选择补数据 / 给预估增量 / 使用0%增量；
6. `年度预计采购量` 始终非空；
7. 输出 `{{项目名称}}-历史测算需求清单.xlsx`。

历史参考价不得写入供方报价字段。

## material-requirement-analysis v0.1.1

### P0 未清零

可以只输出：

- 需求诊断摘要；
- P0/P1/P2；
- 澄清问题；
- 当前阻断原因。

此时允许暂不生成 Excel。

### P0 已清零

**必须生成 Excel，只有文字/YAML/JSON 不算完成。**

如果仍有 P1/P2：

```text
{{项目名称}}-澄清版需求清单.xlsx
```

如果关键需求及项目商务条件均已确认：

```text
{{项目名称}}-最终需求清单.xlsx
```

模板优先级：

1. 用户原始 Excel 的副本，优先保留企业原版式；
2. 当前项目企业模板；
3. `.agents/skills/material-requirement-analysis/templates/标准需求清单模板.xlsx`。

P1/P2 尚未解决时应在备注/待确认事项中留痕，但**不能阻止 Excel 生成**。

供方报价字段如含税单价、未税单价、税率、含税总价应保留为空，不得预填历史价、旧报价或 AI 测算价。

## Codex 测试：material-requirement-analysis

```text
$material-requirement-analysis

分析我的初版需求清单。
先完成 P0/P1/P2 诊断。
我回答 P0 后重新检查：
- 如果 P0 仍 > 0，继续澄清；
- 如果 P0 = 0，必须生成需求清单 Excel；
- 有 P1/P2 时生成“澄清版需求清单.xlsx”；
- 全部关键项确认时生成“最终需求清单.xlsx”；
- 同时输出 Structured Handoff；
- 供方报价字段保持空白。
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 候选供方只能来自企业内部官方供方库。
3. 公网资料不得新增邀标候选供方。
4. 历史采购分析必须先选择协议范围并过滤订单状态。
5. 历史数据不足12个月时形成12个月年化基线。
6. 多区域数量按 `SKU + 区域` 管理。
7. 需求分析 P0 清零后必须生成 Excel。
8. P1/P2 不得阻断澄清版需求清单生成。
9. 需求字段和供方报价字段严格分离。
10. AI计算、历史事实、假设和人工确认必须分开记录。
11. 最终需求数量、候选供方、短名单、预算及定标规则由采购员确认。

## v0.7.5

- `material-requirement-analysis` 升级至 v0.1.1；
- 新增 P0 清零后的 Mandatory Requirement Workbook Output；
- P0=0 且仍有 P1/P2 时输出“澄清版需求清单.xlsx”；
- 关键事项全部确认后输出“最终需求清单.xlsx”；
- 原始 Excel 优先复制并保留版式，不覆盖原文件；
- 新增标准需求清单模板；
- 新增 `references/requirement-workbook-output-rules.md`；
- 只有文字/Structured Handoff 而无 Excel 时，P0 清零后的 Skill 执行视为 incomplete。
