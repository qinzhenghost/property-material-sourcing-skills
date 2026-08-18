# Property Material Sourcing Skills — v0.9.0

物业物资类 AI 邀标寻源 Skill 库。

## Codex 运行入口

```text
.agents/
└── skills/
    ├── historical-procurement-analysis/
    ├── material-requirement-analysis/
    ├── sourcing-invitation/
    ├── supplier-shortlist/
    └── sourcing-strategy/
```

> `official-supplier-matching` 已在 v0.8.0 合并进 `sourcing-invitation`。
>
> `shortlist-approval` 已在 v0.9.0 合并进 `supplier-shortlist`。
>
> `skills/` 暂保留历史兼容镜像；测试和后续开发以 `.agents/skills/` 为主。

## 当前主流程

```text
历史采购数据（可选）
        ↓
historical-procurement-analysis
        ├─ 选择协议编号
        ├─ 仅保留 已完成 / 执行中
        ├─ 不足12个月 → 12个月年度基线
        ├─ 确认年度增量比例
        └─ 历史测算需求清单.xlsx
        ↓
material-requirement-analysis
        ├─ P0 / P1 / P2 需求诊断
        ├─ P0清零
        ├─ 强制生成需求清单.xlsx
        └─ Handoff → sourcing-invitation
        ↓
sourcing-invitation
        ├─ Phase A：官方供方库匹配
        │    ├─ 物资 / 品类 / 区域 / 资质 / 状态 Hard Gate
        │    ├─ 候选池 + evidence + coverage gap
        │    └─ 【采购员确认初版邀标供方】
        └─ Phase B：邀标四件套
             ├─ 01 邀标邮件.eml（单一 BCC）
             ├─ 02 最终需求清单.xlsx（对外附件）
             ├─ 03 招标意向征集登记表.xlsx（对外附件）
             └─ 04 供方信息长名单.xlsx（内部文件）
        ↓
人工审核并发送
        ↓
supplier-shortlist
        ├─ Phase A：供方回复汇总 + 短名单建议
        ├─ 企业供方短名单.xlsx
        ├─ 【采购员确认最终短名单】
        └─ Phase B：短名单报批
             ├─ 最终供方短名单.xlsx
             ├─ 短名单报批邮件.md
             └─ shortlist-handoff.yaml
        ↓
sourcing-strategy
        └─ 采购方案报告.docx
```

## 当前 Skill

| Skill | 作用 | 状态 |
|---|---|---|
| `historical-procurement-analysis` | 协议选择、有效订单过滤、12个月年化、年度增量、历史测算需求清单 | ✅ |
| `material-requirement-analysis` | 需求诊断、P0澄清、强制生成澄清/最终需求清单 Excel | ✅ |
| `sourcing-invitation` | 官方供方匹配 + 人工确认 + 邀标四件套 | ✅ |
| `supplier-shortlist` | 供方回复 → 短名单建议 → 人工确认 → 短名单报批 + strategy_handoff | ✅ |
| `sourcing-strategy` | 生成采购方案/策略报告 | ✅ |

## historical-procurement-analysis v0.5.5

- 用户必须明确选择协议范围；
- 仅 `已完成 / 执行中` 订单进入统计；
- 不足12个月必须折算12个月年度基线；
- 年度预计采购量始终非空；
- 年度增量缺少两年可比数据时由采购员确认补数据 / 预估增量 / 0%增量；
- 输出 `{{项目名称}}-历史测算需求清单.xlsx`；
- 历史参考价不得预填供应商报价字段。

## material-requirement-analysis v0.1.2

P0 清零后必须生成 Excel：

- 有未解决 P1/P2：`{{项目名称}}-澄清版需求清单.xlsx`
- 关键事项均确认：`{{项目名称}}-最终需求清单.xlsx`

下游：

```yaml
next_skill: sourcing-invitation
next_phase: official_supplier_matching
```

## sourcing-invitation v0.4.0

### Phase A — 官方供方匹配

候选供方只能来自企业内部官方供方库。完成 registry tier、品类/区域/资质/状态 Hard Gate、Fit、Evidence、候选池及官方库覆盖缺口后，必须暂停等待采购员确认初版邀标供方。

### Phase B — 邀标四件套

采购员确认初版供方后固定生成：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
04 {{项目名称}}-供方信息长名单.xlsx
```

- 01：单一 EML，供方邮箱仅放 BCC，不自动发送；
- 02、03：实际对外附件；
- 03：必须复制企业原版模板；`N3` 保证金 = `CEILING(采购清单预估总金额 × 1%, 1000)`；
- 04：采购内部文件，只列人工确认初版供方及其已确认联系人/电话/邮箱，不外发。

## supplier-shortlist v0.5.0

### Phase A — 供方回复与短名单草稿

输入 `sourcing-invitation` 已确认初版供方长名单和供方实际回复，使用企业 `供方短名单模板.xlsx` 汇总：

- 头部企业合作业绩；
- 工厂 / 代理 / 资质 / 仓储 / 运输等能力；
- 公司性质；
- 合作意向；
- 合作历史；
- 注册资本；
- 供方库分类；
- 邮箱 / 电话；
- AI建议：建议入围 / 待澄清 / 不建议入围。

生成：

`{{项目名称}}-供方短名单.xlsx`

然后必须暂停等待采购员确认。AI建议不等于最终短名单。

### Phase B — 短名单报批

采购员确认后，同一 Skill：

1. 固化最终 `{{项目名称}}-供方短名单.xlsx`；
2. 生成 `{{项目名称}}-短名单报批邮件.md`；
3. 生成 `{{项目名称}}-shortlist-handoff.yaml`；
4. 下游进入 `sourcing-strategy`。

Phase B 不允许重新评分、重新排序或改变采购员确认结果。

## sourcing-strategy v0.7.1

直接消费：

- confirmed requirement；
- `sourcing-invitation` 的官方库覆盖 / 保证金等可追溯结果；
- `supplier-shortlist.strategy_handoff` 的人工确认最终短名单；
- 历史采购分析（如有）；
- 当前项目采购规则。

生成：

```text
{{项目名称}}-strategy-data.yaml
{{项目名称}}-采购方案报告.docx
```

## 核心原则

1. V1 只做物业物资类采购，不做服务类。
2. 候选供方身份只能来自企业内部官方供方库。
3. 公网资料不得新增邀标候选供方。
4. AI候选供方与采购员确认初版供方严格分离。
5. AI短名单建议与采购员最终短名单严格分离。
6. Human Gate 放在 Skill 内部，不因合并而取消。
7. 需求分析 P0 清零后必须生成 Excel。
8. 需求字段与供方报价字段严格分离。
9. 邀标邮件单一 EML + BCC，不自动发送。
10. 供方信息长名单仅采购内部使用。
11. 招标意向征集登记表必须复制企业原版模板生成。
12. N3 投标保证金按当前项目预估总金额动态计算。
13. Phase B 报批不得重新改变采购员确认的最终短名单。
14. sourcing-strategy 必须消费人工确认的 shortlist_handoff，而不是 Phase A AI建议。
15. 所有重要结论、统计和计算必须可追溯。

## v0.9.0

- 将 `shortlist-approval` 合并进 `supplier-shortlist`；
- `supplier-shortlist` 升级至 v0.5.0，拆分 Phase A / Phase B；
- Phase A 生成企业短名单表草稿并强制等待采购员确认；
- Phase B 固化最终短名单并生成报批邮件 + strategy_handoff；
- 删除独立 `shortlist-approval` Skill 入口；
- 新增统一 `schemas/supplier-shortlist.schema.yaml`；
- 删除旧 `schemas/shortlist-approval.schema.yaml`；
- `sourcing-strategy` 升级至 v0.7.1，只消费合并后的人工确认 shortlist_handoff；
- 主流程压缩为 5 个 Skill。
