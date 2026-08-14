# Property Material Sourcing Skills — v0.6.0

物业物资类 AI 邀标寻源 Skill 库。

## 当前流程

```text
历史采购数据（可选）
        ↓
historical-procurement-analysis
        ↓
初版需求 → material-requirement-analysis → 标准需求
        ↓
official-supplier-matching
        ↓
人工确认沟通供方
        ↓
sourcing-invitation
        ↓
邀标邮件 + 最终需求清单 + 招标意向征集登记表
        ↓
supplier-shortlist
        ↓
供方短名单表
        ↓
人工确认短名单
        ↓
shortlist-approval
        ↓
短名单报批邮件 + strategy_handoff
        ↓
sourcing-strategy（下一阶段）
```

如果项目没有历史采购数据，可直接跳过 `historical-procurement-analysis`。

## 当前 Skill

```text
skills/
├── historical-procurement-analysis/
├── material-requirement-analysis/
├── official-supplier-matching/
├── sourcing-invitation/
├── supplier-shortlist/
└── shortlist-approval/
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 采购制度为邀标制。
3. 候选供方只能来自企业内部官方供方库或其明确授权的官方导出。
4. 公网搜索、行业榜单、搜索引擎、第三方企业库不得新增邀标候选供方。
5. 有历史采购数据时，先形成可追溯的历史采购基线，再进入需求澄清。
6. 历史数量预测必须保留历史基准、增长率来源、计算公式、可信度和人工确认状态。
7. 历史数据覆盖多个区域时，数量必须按 `SKU + 区域` 聚合。
8. 邀标阶段固定输出：邀标邮件正文 + 最终需求清单 + 招标意向征集登记表。
9. 供方回复阶段只做轻量短名单汇总，不做复杂评分或报价模型。
10. `shortlist-approval` 不重新评价供应商，只基于采购员确认后的短名单生成报批邮件和后续策略 Handoff。
11. 最终需求数量、候选供方、短名单及审批结论等关键决策由采购员确认。

## v0.6.0

新增 `shortlist-approval`：

- 读取采购员确认后的《供方短名单》；
- 汇总长名单、沟通、回复、入围、不入围和待澄清数量；
- 生成短名单报批邮件草稿；
- 保留真实入围/未入围原因；
- 不重新评分、不改变短名单；
- 生成 `strategy_handoff`，供后续采购策略报告直接使用；
- 邮件只生成草稿，不自动发送。

完整项目文件由本仓库持续维护。
