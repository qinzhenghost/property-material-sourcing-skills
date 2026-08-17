# Property Material Sourcing Skills — v0.8.0

物业物资类 AI 邀标寻源 Skill 库。

## Codex 运行入口

```text
.agents/
└── skills/
    ├── historical-procurement-analysis/
    ├── material-requirement-analysis/
    ├── sourcing-invitation/
    ├── supplier-shortlist/
    ├── shortlist-approval/
    └── sourcing-strategy/
```

> `official-supplier-matching` 已在 v0.8.0 合并进 `sourcing-invitation`，不再作为独立 Skill。
>
> `skills/` 暂保留历史兼容镜像；测试和后续开发以 `.agents/skills/` 为主。

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
        └─ Handoff → sourcing-invitation
        ↓
sourcing-invitation
        ├─ Phase A：仅从企业官方供方库匹配
        ├─ 品类 / 区域 / 资质 / 状态 Hard Gate
        ├─ 候选供方池 + evidence + 官方库覆盖缺口
        ├─ 【采购员确认初版邀标供方】
        └─ Phase B：生成邀标四件套
             ├─ 01 邀标邮件.eml（单一 BCC）
             ├─ 02 最终需求清单.xlsx（对外附件）
             ├─ 03 招标意向征集登记表.xlsx（对外附件）
             └─ 04 供方信息长名单.xlsx（内部文件）
        ↓
人工审核并实际添加 02、03 后发送
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
| `sourcing-invitation` | 官方供方匹配 + 人工确认 + 邀标四件套 | ✅ |
| `supplier-shortlist` | 根据供方回复生成短名单表 | ✅ |
| `shortlist-approval` | 短名单报批邮件及策略 Handoff | ✅ |
| `sourcing-strategy` | 生成采购方案/策略报告 | ✅ |

## historical-procurement-analysis v0.5.5

历史分析必须依次经过：

1. 用户明确选择一个或多个协议编号；
2. 仅保留 `已完成 / 执行中` 订单；
3. 不足12个月按有效覆盖周期折算12个月年度基线；
4. 有两年度可比数据时计算年度增量；
5. 无两年度可比数据时由用户选择补数据 / 给预估增量 / 使用0%增量；
6. `年度预计采购量` 始终非空；
7. 输出 `{{项目名称}}-历史测算需求清单.xlsx`。

历史参考价不得写入供方报价字段。

## material-requirement-analysis v0.1.2

P0 未清零时可以只输出诊断与澄清问题。

P0 清零后必须生成 Excel：

- 仍有 P1/P2：`{{项目名称}}-澄清版需求清单.xlsx`
- 关键事项全部确认：`{{项目名称}}-最终需求清单.xlsx`

下游统一为：

```text
next_skill: sourcing-invitation
next_phase: official_supplier_matching
```

供方报价字段保持空白。

## sourcing-invitation v0.4.0

### Phase A — 官方供方匹配

候选供方只能来自企业内部官方供方库。

必须完成：

- 官方数据源与 registry tier 登记；
- 物资类 Gate；
- 二级/三级品类匹配；
- 区域 full / partial / no_match / unknown；
- 资质与官方状态检查；
- Hard Gate；
- Fit 辅助分析；
- 候选池分组；
- 官方库覆盖缺口检查；
- 每个结论的 evidence。

然后强制暂停等待采购员确认：

```text
include_for_intention_collection
exclude
hold
request_more_official_data
```

AI候选池不等于正式邀标供方。

### Phase B — 邀标四件套

采购员确认初版供方后固定生成：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
04 {{项目名称}}-供方信息长名单.xlsx
```

#### 01 邮件规则

- 每个项目只生成 1 个 EML；
- 已确认且邮箱已确认的供方统一写入 `Bcc`；
- 供方邮箱不得进入 `To` / `Cc`；
- `To` 仅允许采购方本方发送/归档邮箱，否则留空；
- `Cc` 仅允许采购员确认的内部邮箱；
- EML 不嵌入附件；
- 正文只引用 02、03；
- 不自动发送。

#### 03 招标意向征集登记表

必须复制企业原版模板生成，不得重建或重新设计。

模板：

```text
.agents/skills/sourcing-invitation/templates/招标意向征集登记表模板.xlsx
```

`Sheet1!N3` 投标保证金：

```text
raw = 采购清单预估总金额 × 1%
投标保证金 = CEILING(raw, 1000)
```

即向上取整到 1000 元整数倍。

缺少可追溯预估总金额时不得沿用历史固定金额。

#### 04 供方信息长名单

只覆盖采购员已确认的初版供方，至少列：

- 供方名称
- 供方编码
- 联系人姓名
- 联系电话
- 邮箱地址
- 联系信息来源
- 联系信息状态
- 备注

联系人/电话/邮箱只能来自官方供方库或采购员明确确认；缺失留空并标记，不得猜测。

04 为采购内部文件，不得发送给供方。

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 候选供方只能来自企业内部官方供方库。
3. 公网资料不得新增邀标候选供方。
4. AI候选池与采购员确认的初版供方必须分离。
5. 历史采购分析必须先选择协议范围并过滤订单状态。
6. 历史数据不足12个月时形成12个月年化基线。
7. 多区域数量按 `SKU + 区域` 管理。
8. 需求分析 P0 清零后必须生成 Excel。
9. P1/P2 不得阻断澄清版需求清单生成。
10. 需求字段与供方报价字段严格分离。
11. 邀标邮件只生成一个 EML，并使用 BCC 隐藏供方邮箱。
12. EML 不嵌入附件；02、03 由采购员发送前人工添加。
13. 04 供方信息长名单仅内部使用。
14. 招标意向征集登记表必须复制企业原版模板生成。
15. N3 投标保证金按项目预估总金额动态计算。
16. AI计算、历史事实、假设和人工确认必须分开记录。
17. 最终需求、初版邀标供方、短名单、预算及定标规则由采购员确认。

## v0.8.0

- 将 `official-supplier-matching` 完整合并进 `sourcing-invitation`；
- `sourcing-invitation` 升级至 v0.4.0，内部拆分 Phase A / Phase B；
- 保留官方供方库 ONLY、Hard Gate、evidence、coverage gap 和采购员人工确认；
- 删除独立 `official-supplier-matching` Skill 入口；
- `material-requirement-analysis` 下游改为直接进入 `sourcing-invitation` Phase A；
- 邀标固定交付升级并固化为四件套，包含内部供方信息长名单；
- `sourcing-invitation-package.schema.yaml` 升级至 v0.4.0，统一承载供方匹配与邀标包；
- 原 `supplier-match.schema.yaml` 不再作为独立 Skill Handoff Schema；
- `.agents/skills/` 与兼容 `skills/` 同步更新。
