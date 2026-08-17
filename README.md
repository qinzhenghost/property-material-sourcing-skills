# Property Material Sourcing Skills — v0.7.1

物业物资类 AI 邀标寻源 Skill 库。

## Codex 运行入口

本仓库已适配 Codex 仓库级 Skill 发现规则。

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

使用 Codex 打开本仓库后，可通过 `/skills` 或显式 `$skill-name` 选择子 Skill 进行测试。

> `skills/` 目录暂时保留用于历史兼容；从 v0.7.1 起，Codex 测试和后续开发以 `.agents/skills/` 为主入口。v1.0 稳定后再决定是否移除旧目录。

## 当前流程

```text
历史采购数据（可选）
        ↓
historical-procurement-analysis
        ↓
初版需求 → AI需求诊断/澄清 → 标准需求
        ↓
官方供方库匹配
        ↓
人工确认沟通供方
        ↓
生成邀标邮件 + 最终需求清单 + 招标意向征集登记表
        ↓
汇总供方回复
        ↓
生成供方短名单表
        ↓
采购员确认短名单
        ↓
shortlist-approval
        ↓
短名单报批邮件 + strategy_handoff
        ↓
sourcing-strategy
        ↓
采购方案报告.docx
```

如果项目没有历史采购数据，可直接跳过 `historical-procurement-analysis`，进入 `material-requirement-analysis`。

## 当前 Skill

| Skill | 作用 | 状态 |
|---|---|---|
| `historical-procurement-analysis` | 历史采购量、金额、价格、区域及需求基线分析 | ✅ |
| `material-requirement-analysis` | 需求诊断、缺失项及标准需求 | ✅ |
| `official-supplier-matching` | 仅从官方供方库匹配候选供方 | ✅ |
| `sourcing-invitation` | 生成邀标三件套 | ✅ |
| `supplier-shortlist` | 根据供方回复生成短名单表 | ✅ |
| `shortlist-approval` | 短名单报批邮件及策略 Handoff | ✅ |
| `sourcing-strategy` | 生成采购方案/策略报告 | ✅ |

## Codex 单 Skill 测试

建议测试阶段全部显式调用，避免把自动路由问题和 Skill 本身的问题混在一起。

例如：

```text
$material-requirement-analysis

请分析 test-data/初版需求清单.xlsx。
本次只测试需求分析，不调用其他采购 Skill。
```

历史采购测试：

```text
$historical-procurement-analysis

分析这份历史订单导出：
1. 判断数据周期是否完整；
2. 按 SKU / 区域聚合；
3. 部分周期禁止机械年化；
4. 输出 requirement_handoff 和 strategy_report_handoff。
```

每个 Skill 建议至少测试：

1. 正常案例；
2. 缺失数据案例；
3. 越权/Guardrail 反例。

## 项目目录

```text
property-material-sourcing-skills/
├── .agents/
│   └── skills/                 # Codex 主运行入口
│       ├── historical-procurement-analysis/
│       ├── material-requirement-analysis/
│       ├── official-supplier-matching/
│       ├── sourcing-invitation/
│       ├── supplier-shortlist/
│       ├── shortlist-approval/
│       └── sourcing-strategy/
├── skills/                     # v0.7.1 暂保留的历史兼容目录
├── schemas/                    # 各阶段结构化数据契约
├── examples/                   # 黄金案例 / Handoff 示例
├── README.md
└── CHANGELOG.md
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 采购制度为邀标制。
3. 候选供方只能来自企业内部官方供方库或其明确授权的官方导出。
4. 公网搜索、行业榜单、搜索引擎、第三方企业库不得新增邀标候选供方。
5. 有历史采购数据时，优先形成可追溯的历史采购基线，再进入需求澄清。
6. 历史数量预测必须保留历史基准、增长率来源、公式、可信度和人工确认状态。
7. 采购规模增长与价格增长必须分开计算，禁止重复增加预计金额。
8. 部分月份/周度历史数据可做汇总和价格基线，但不得默认年化。
9. 多区域历史采购量必须按 `SKU + 区域` 聚合；重复需求行未标区域时禁止自动回填数量。
10. 邀标阶段固定输出：邀标邮件正文 + 最终需求清单 + 招标意向征集登记表。
11. 供方回复阶段只做轻量短名单汇总，不做复杂评分或报价模型。
12. 短名单报批阶段只整理采购员已确认结果，不重新评价供应商。
13. `sourcing-strategy` 只整合前序已确认数据，不重新发明需求、预算、短名单或定标规则。
14. 公开市场信息仅可用于供应市场行情、成本、价格趋势和政策分析，不得污染官方供方候选池。
15. 模板中的示例值不得自动继承到新项目。
16. 最终需求数量、候选供方、短名单、预算、保证金、目标价格、定标规则和评标人员等关键决策由采购员确认。

## v0.7.1

- 新增 `.agents/skills/` Codex 原生运行入口；
- 7 个现有子 Skill 已完整映射到该目录；
- `skills/` 暂时保留兼容；
- README 增加安装与单 Skill 测试说明；
- 下一阶段先做子 Skill 实测和回归测试，再进入 Parent Router / v1.0。
