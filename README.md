# Property Material Sourcing Skills — v0.5.0

物业物资类 AI 邀标寻源 Skill 库。

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
```

如果项目没有历史采购数据，可直接跳过 `historical-procurement-analysis`，进入 `material-requirement-analysis`。

## 当前 Skill

```text
skills/
├── historical-procurement-analysis/
├── material-requirement-analysis/
├── official-supplier-matching/
├── sourcing-invitation/
└── supplier-shortlist/
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 采购制度为邀标制。
3. 候选供方只能来自企业内部官方供方库或其明确授权的官方导出。
4. 公网搜索、行业榜单、搜索引擎、第三方企业库不得新增邀标候选供方。
5. 有历史采购数据时，优先形成可追溯的历史采购基线，再进入需求澄清。
6. 历史数量预测必须保留：历史基准、增长率来源、计算公式、可信度和人工确认状态。
7. 采购规模增长与价格增长必须分开计算，禁止重复增加预计金额。
8. 历史分析结果同时服务：
   - 需求清单中的预计采购量/预计金额；
   - 后续采购策略报告中的支出分析、历史趋势和需求测算依据。
9. 邀标阶段固定输出：
   - 邀标/意向征集邮件正文；
   - 最终确认版需求清单；
   - 招标意向征集登记表。
10. 供方回复阶段只做轻量短名单汇总，不做复杂评分或报价模型。
11. 最终需求数量、候选供方、短名单等关键决策由采购员确认。

## v0.5.0

新增 `historical-procurement-analysis`：

- 读取历史采购量、金额、单价及采购规模数据；
- 识别 SKU 可比性、单位/税口径和异常采购；
- 计算同比采购趋势；
- 在有明确采购规模增幅时，按可解释公式生成下一周期建议采购量/金额；
- 生成 `requirement_handoff`，用于回填需求清单数量；
- 生成 `strategy_report_handoff`，用于后续采购策略报告。

完整项目文件由本仓库持续维护。
