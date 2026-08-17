# Property Material Sourcing Skills — v0.7.3

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
        ├─ SKU + 区域 历史数量/金额/价格分析
        ├─ 条件满足时测算下一周期数量/金额
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
| `historical-procurement-analysis` | 协议范围选择、有效订单过滤、历史采购分析、需求数量基线、历史测算需求清单 | ✅ |
| `material-requirement-analysis` | 需求诊断、缺失项及标准需求 | ✅ |
| `official-supplier-matching` | 仅从官方供方库匹配候选供方 | ✅ |
| `sourcing-invitation` | 生成邀标三件套 | ✅ |
| `supplier-shortlist` | 根据供方回复生成短名单表 | ✅ |
| `shortlist-approval` | 短名单报批邮件及策略 Handoff | ✅ |
| `sourcing-strategy` | 生成采购方案/策略报告 | ✅ |

## historical-procurement-analysis v0.5.4

历史订单进入正式分析前必须经过两道 Gate。

### Gate 1：协议编号选择

先识别订单文件中的全部唯一协议编号，并展示每个协议的：

- 原始订单数；
- 有效订单数；
- 有效明细行；
- 数据期间；
- SKU 数；
- 有效含税金额；
- 主要区域。

然后由用户明确选择一个或多个协议编号。

禁止：

- 默认选择最新协议；
- 默认选择金额最大的协议；
- 默认选择全部协议；
- 即使只有一个协议，也不能跳过用户确认。

无协议编号订单默认排除，只有用户明确确认后才允许纳入。

### Gate 2：订单状态过滤

正式分析只保留：

```text
已完成
执行中
```

剔除已取消、已退货、取消、退货、已作废、关闭及其他非有效状态。

## 历史测算需求清单

固定输出：

```text
{{项目名称}}-历史测算需求清单.xlsx
```

模板：

```text
.agents/skills/historical-procurement-analysis/templates/需求清单模板.xlsx
```

历史分析可回填区域、名称、品牌、规格、单位和有可靠依据的年度预计采购量。

以下供方报价字段保持空白：

- 含税单价
- 未税单价
- 税率
- 含税总价

## Codex 单 Skill 测试示例

```text
$historical-procurement-analysis

分析 test-data/历史订单.xls。
本次只测试 historical-procurement-analysis：
1. 识别协议编号/合同编号字段；
2. 列出全部协议编号及每个协议的有效订单概览；
3. 在我明确选择协议前，不做正式历史分析；
4. 选择后，只分析所选协议；
5. 仅保留“已完成”“执行中”订单；
6. 输出被剔除状态及数量；
7. 按 SKU + 区域汇总历史采购量/金额；
8. 判断周期是否支持年度预测；
9. 输出 requirement_handoff 和 strategy_report_handoff；
10. 按模板生成历史测算需求清单.xlsx；
11. 供方报价字段保持空白。
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 采购制度为邀标制。
3. 候选供方只能来自企业内部官方供方库。
4. 公网资料不得新增候选供方。
5. 历史分析必须先由用户明确选择协议范围。
6. 历史订单仅统计“已完成/执行中”。
7. 部分周期数据不得默认年化。
8. 多区域数量按 `SKU + 区域` 聚合。
9. 采购规模增长与价格增长分开计算。
10. AI 计算、历史事实和人工确认必须分开记录。
11. 最终需求数量、候选供方、短名单、预算及定标规则由采购员确认。

## v0.7.3

- `historical-procurement-analysis` 升级至 v0.5.4；
- 新增强制 Agreement Selection Gate；
- 用户必须明确选择一个或多个协议编号后才能继续；
- 多协议分析保留协议维度并先分别统计再合并；
- 无协议编号订单默认排除；
- 协议选择与订单状态过滤均进入结构化 Handoff；
- historical-procurement Schema 升级至 0.5.4；
- 新增 `agreement-selection-rules.md`；
- 重写主 Skill 为标准 UTF-8，修复历史 Git Blob 编码异常。
