# Property Material Sourcing Skills — v0.7.4

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
        ├─ 识别全部协议编号
        ├─ 用户明确选择一个或多个协议编号
        ├─ 只保留所选协议数据
        ├─ 仅保留 已完成 / 执行中 订单
        ├─ 剔除 已取消 / 已退货 / 其他状态
        ├─ 不足12个月 → 强制折算为12个月年化基线
        ├─ 有两年度可比数据 → 计算年度增量比例
        ├─ 无两年度可比数据 → 用户选择补充数据 / 给预估增量 / 0%增量
        ├─ 年度预计采购量始终非空
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
| `historical-procurement-analysis` | 协议范围选择、有效订单过滤、12个月年化、年度增量确认、历史测算需求清单 | ✅ |
| `material-requirement-analysis` | 需求诊断、缺失项及标准需求 | ✅ |
| `official-supplier-matching` | 仅从官方供方库匹配候选供方 | ✅ |
| `sourcing-invitation` | 生成邀标三件套 | ✅ |
| `supplier-shortlist` | 根据供方回复生成短名单表 | ✅ |
| `shortlist-approval` | 短名单报批邮件及策略 Handoff | ✅ |
| `sourcing-strategy` | 生成采购方案/策略报告 | ✅ |

## historical-procurement-analysis v0.5.5

历史订单进入正式分析前依次经过：

### Gate 1：协议编号选择

先识别订单文件中的全部唯一协议编号，并展示每个协议的原始订单数、有效订单数、有效明细行、数据期间、SKU数、有效含税金额和主要区域，然后由用户明确选择一个或多个协议编号。

禁止默认选择最新协议、金额最大的协议或全部协议；即使只有一个协议也必须确认。

### Gate 2：订单状态过滤

正式分析只保留：

```text
已完成
执行中
```

剔除已取消、已退货、取消、退货、已作废、关闭及其他非有效状态。

### Gate 3：12个月年化

如果所选协议的有效历史数据不足12个月，必须按覆盖周期比例折算为12个月采购基线。

完整自然月数据：

```text
12个月年化基线 = 有效历史采购量 × 12 / 覆盖月数
```

首尾月份不完整时，按有效数据起止日期换算月当量后再折算。

年化结果属于 `calculated`，不是历史全年实际；必须提示季节性、集中采购和一次性项目风险。

### Gate 4：年度增量比例

优先依据：

1. 可比两年度数量同比；
2. 可比两年度采购金额同比，作为采购规模代理；
3. 企业正式业务规模/预算增幅；
4. 用户确认的预估增量比例；
5. 用户确认0%增量。

如果没有两年度可比采购金额/数量，Skill 必须让用户选择：

- 补充另一年度同口径数据；
- 给出预估年度增量比例；
- 按0%增量。

在用户尚未确认增量前，`年度预计采购量` 也不能留空：先填入12个月年化基线，并标记“当前为12个月年化基线，年度增量比例待确认”。

用户确认后：

```text
最终年度预计采购量
=
12个月年化基线 × (1 + 确认的年度增量比例)
```

采购金额同比可能包含价格变化，因此不得把同一个金额同比同时用于数量增长和价格增长。

## 历史测算需求清单

固定输出：

```text
{{项目名称}}-历史测算需求清单.xlsx
```

模板：

```text
.agents/skills/historical-procurement-analysis/templates/需求清单模板.xlsx
```

`年度预计采购量` 必须有值。

以下供方报价字段始终保持空白：

- 含税单价
- 未税单价
- 税率
- 含税总价

## Codex 单 Skill 测试示例

```text
$historical-procurement-analysis

分析 test-data/历史订单.xls。
本次只测试 historical-procurement-analysis：
1. 识别协议编号并让我选择一个或多个协议；
2. 只保留所选协议；
3. 只保留已完成/执行中订单；
4. 统计有效数据覆盖周期；
5. 不足12个月时按比例折算为12个月采购量；
6. 如果有两年度可比数据，计算年度同比增量；
7. 如果没有两年度可比数据，提醒我选择补充数据、给预估增量比例或按0%增量；
8. 年度预计采购量不得留空；
9. 生成历史测算需求清单.xlsx；
10. 供方报价字段保持空白。
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 采购制度为邀标制。
3. 候选供方只能来自企业内部官方供方库。
4. 公网资料不得新增候选供方。
5. 历史分析必须先由用户明确选择协议范围。
6. 历史订单仅统计“已完成/执行中”。
7. 不足12个月历史数据必须折算为12个月年化基线。
8. 年度预计采购量不得留空。
9. 年度增量必须有两年度可比数据、正式业务依据或用户确认。
10. 多区域数量按 `SKU + 区域` 聚合。
11. 数量增长与价格增长分开计算。
12. AI计算、历史事实和人工确认必须分开记录。
13. 最终需求数量、候选供方、短名单、预算及定标规则由采购员确认。

## v0.7.4

- `historical-procurement-analysis` 升级至 v0.5.5；
- 将原“部分周期不年化”规则调整为“部分周期必须折算12个月基线”；
- 新增 `annualization` 结构，记录覆盖周期、月当量和年化系数；
- 新增 `growth_rate_decision`，处理两年度同比或用户确认增量；
- 缺少两年度可比数据时，强制提示补数据 / 给预估增量 / 0%增量三选一；
- 增量未确认时，需求清单先填12个月年化基线，不再留空；
- 增加 `provisional_annualized_baseline` 与 `final_projected` 状态；
- 新增 `references/annualization-and-growth-rules.md`；
- Schema 与历史采购示例同步升级至 0.5.5。
