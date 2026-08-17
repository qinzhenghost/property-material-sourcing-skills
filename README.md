# Property Material Sourcing Skills — v0.7.8

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
        ├─ 官方供方库匹配
        └─ 采购员确认邀标沟通供方
        ↓
sourcing-invitation
        ├─ 1个 BCC 群发邀标邮件.eml
        ├─ 最终需求清单.xlsx（独立交付）
        └─ 招标意向征集登记表.xlsx（企业原版模板副本生成）
             └─ N3 投标保证金 = 采购清单预估总金额 × 1%，向上取整到1000元整数倍
        ↓
人工审核并实际添加两个附件后发送
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
| `sourcing-invitation` | 生成单一 BCC EML + 两个独立 Excel；登记表严格复用企业原版模板并动态计算 N3 保证金 | ✅ |
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

P0 未清零时可以只输出诊断与澄清问题。

P0 清零后必须生成 Excel：

- 仍有 P1/P2：`{{项目名称}}-澄清版需求清单.xlsx`
- 关键事项全部确认：`{{项目名称}}-最终需求清单.xlsx`

模板优先级：

1. 用户原始 Excel 的副本；
2. 当前项目企业模板；
3. `.agents/skills/material-requirement-analysis/templates/标准需求清单模板.xlsx`。

供方报价字段保持空白。

## sourcing-invitation v0.3.3

固定只生成三个独立文件：

```text
{{项目名称}}-邀标邮件.eml
{{项目名称}}-最终需求清单.xlsx
{{项目名称}}-招标意向征集登记表.xlsx
```

邮件规则：

- 只生成 1 个 EML；
- 所有人工确认且邮箱已确认的供方统一放入 `Bcc`；
- 供方不得出现在 `To` 或 `Cc`；
- 缺失/冲突的供方邮箱不得猜测；
- `To` 仅可使用采购方明确提供的本方发送/归档邮箱，否则留空；
- `Cc` 仅使用采购员明确确认的内部邮箱；
- EML 不嵌入两个 Excel；
- 两个 Excel 单独交付，由采购员实际发送前人工添加为附件；
- 不自动发送邮件。

### 招标意向征集登记表原模板规则

- 仓库模板必须与企业原始附件保持相同结构和样式；
- 生成 03 文件时必须复制原模板，再填入允许的当前项目变量；
- 禁止新建“类似模板”、重新排版或用通用表格替代；
- 工作表、行列结构、合并单元格、样式、打印设置、公式/验证及供方填写区域必须保持；
- 供方待填写字段默认保持空白。

企业模板位置：

```text
.agents/skills/sourcing-invitation/templates/招标意向征集登记表模板.xlsx
```

### N3 投标保证金变量

企业模板 `Sheet1!N3` 使用：

```text
{{投标保证金金额}}
```

运行时计算：

```text
投标保证金原始值 = 采购清单预估总金额 × 1%
投标保证金金额 = CEILING(投标保证金原始值, 1000)
```

即向上取整到 1000 元整数倍。

示例：

```text
采购清单预估总金额：523,673.81 元
1%：5,236.7381 元
N3 投标保证金：6,000 元
```

规则：

- 采购清单预估总金额必须可追溯；
- 不得从供应商未填写的报价列反推；
- 不得沿用历史模板固定 6000 元；
- 缺少预估总金额时，03 登记表必须提示补充/确认，不能默认金额；
- 项目文件生成后 N3 不得残留 `{{投标保证金金额}}`。

规则文件：

```text
.agents/skills/sourcing-invitation/references/bid-bond-variable-rules.md
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
10. 邀标邮件只生成一个 EML，并使用 BCC 隐藏供方邮箱。
11. 邀标 EML 不嵌入附件，两个 Excel 独立交付。
12. 招标意向征集登记表必须复制企业原版模板生成，不得重建。
13. N3 投标保证金按项目预估总金额动态计算，不允许继承历史固定值。
14. AI计算、历史事实、假设和人工确认必须分开记录。
15. 最终需求数量、候选供方、短名单、预算及定标规则由采购员确认。

## v0.7.8

- `sourcing-invitation` 升级至 v0.3.3；
- 企业《招标意向征集登记表》`Sheet1!N3` 的固定保证金改为 `{{投标保证金金额}}` 变量；
- 保证金规则固定为 `采购清单预估总金额 × 1%`；
- 计算结果向上取整到 1000 元整数倍；
- 新增采购清单预估总金额来源与可追溯要求；
- 缺失预估总金额时不得使用默认金额，03 登记表进入阻断/待补充状态；
- 新增 `references/bid-bond-variable-rules.md`；
- `sourcing-invitation-package.schema.yaml` 升级至 v0.3.3；
- 企业模板 N3 已改为变量占位符，`.agents/skills/` 与兼容 `skills/` 同步更新。
