# Property Material Sourcing Skills — v0.7.2

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

使用 Codex 打开本仓库后，可通过 `/skills` 或 `$skill-name` 显式调用子 Skill。

> `skills/` 暂时保留兼容；后续开发和测试以 `.agents/skills/` 为主。

## 当前流程

```text
历史采购数据（可选）
        ↓
historical-procurement-analysis
        ├─ 仅保留 已完成 / 执行中 订单
        ├─ 历史数量/金额/价格/区域分析
        ├─ 下一周期数量/金额测算
        └─ 输出历史测算需求清单.xlsx
        ↓
material-requirement-analysis
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

没有历史采购数据时，可直接跳过 `historical-procurement-analysis`。

## 当前 Skill

| Skill | 作用 | 状态 |
|---|---|---|
| `historical-procurement-analysis` | 有效历史订单过滤、历史采购分析、需求数量基线、历史测算需求清单 | ✅ |
| `material-requirement-analysis` | 需求诊断、缺失项及标准需求 | ✅ |
| `official-supplier-matching` | 仅从官方供方库匹配候选供方 | ✅ |
| `sourcing-invitation` | 生成邀标三件套 | ✅ |
| `supplier-shortlist` | 根据供方回复生成短名单表 | ✅ |
| `shortlist-approval` | 短名单报批邮件及策略 Handoff | ✅ |
| `sourcing-strategy` | 生成采购方案/策略报告 | ✅ |

## historical-procurement-analysis v0.5.3

历史订单进入任何汇总前必须先做状态过滤：

```text
保留：已完成、执行中
剔除：已取消、已退货及所有其他状态
```

如果订单导出没有状态字段，不得默认全部有效；只有采购员明确确认数据已预过滤后才能继续形成正式基线。

固定新增输出：

```text
{{项目名称}}-历史测算需求清单.xlsx
```

模板位置：

```text
.agents/skills/historical-procurement-analysis/templates/需求清单模板.xlsx
```

需求清单显式包含“区域”列。历史分析可回填采购方需求字段，但以下供方报价字段必须保持空白：

- 含税单价
- 未税单价
- 税率
- 含税总价

历史参考单价/预计采购金额保留在分析结果和 Handoff 中。

## Codex 单 Skill 测试示例

```text
$historical-procurement-analysis

分析 test-data/历史订单.xls。
本次只测试 historical-procurement-analysis：
1. 先识别订单状态字段；
2. 只保留“已完成”“执行中”；
3. 输出被剔除状态及数量；
4. 再按 SKU + 区域汇总历史采购量/金额；
5. 判断周期是否支持年度预测；
6. 输出 requirement_handoff 和 strategy_report_handoff；
7. 按需求清单模板生成“历史测算需求清单.xlsx”；
8. 供方报价字段必须保持空白。
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 采购制度为邀标制。
3. 候选供方只能来自企业内部官方供方库。
4. 公网资料不得新增候选供方。
5. 历史订单仅统计“已完成/执行中”。
6. 部分周期数据不得默认年化。
7. 多区域数量按 `SKU + 区域` 聚合。
8. 采购规模增长与价格增长分开计算。
9. AI 计算、历史事实和人工确认必须分开记录。
10. 最终需求数量、候选供方、短名单、预算及定标规则由采购员确认。

## v0.7.2

- `historical-procurement-analysis` 升级至 v0.5.3；
- 新增强制订单状态过滤 Gate；
- 仅保留“已完成/执行中”订单；
- 新增过滤审计摘要；
- 新增标准需求清单 Excel 固定输出；
- 新增带“区域”字段的需求清单模板；
- 历史参考价不得写入供方报价列；
- 更新 historical-procurement Schema 与示例。
