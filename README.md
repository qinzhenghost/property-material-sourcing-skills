# Property Material Sourcing Skills — v0.9.1

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
        ├─ 区域/业务单元/订单金额段/SKU/供方/价格支出结构
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
        ├─ 深度支出分析
        ├─ 当地供应市场行情研究
        ├─ 短名单/定标/风险逻辑整合
        └─ 采购方案报告.docx
```

## 当前 Skill

| Skill | 作用 | 状态 |
|---|---|---|
| `historical-procurement-analysis` | 协议选择、有效订单过滤、12个月年化、年度增量、多维历史支出 Handoff | ✅ |
| `material-requirement-analysis` | 需求诊断、P0澄清、强制生成澄清/最终需求清单 Excel | ✅ |
| `sourcing-invitation` | 官方供方匹配 + 人工确认 + 邀标四件套 | ✅ |
| `supplier-shortlist` | 供方回复 → 短名单建议 → 人工确认 → 短名单报批 + strategy_handoff | ✅ |
| `sourcing-strategy` | 深度支出/市场分析 + 采购方案/策略报告 | ✅ |

## historical-procurement-analysis v0.5.6

- 用户必须明确选择协议范围；
- 仅 `已完成 / 执行中` 订单进入统计；
- 不足12个月必须折算12个月年度基线；
- 年度预计采购量始终非空；
- 年度增量缺少两年可比数据时由采购员确认补数据 / 预估增量 / 0%增量；
- 有数据时向策略报告 Handoff 提供：整体、区域、业务单元/军种/业态/项目、订单金额段、SKU/Pareto、历史供方集中度、价格分析；
- 订单编号 + 订单金额存在时必须分析订单碎片化，为“免运额度/最低起送”提供依据；
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

## sourcing-invitation v0.4.x

### Phase A — 官方供方匹配

候选供方只能来自企业内部官方供方库。完成 registry tier、品类/区域/资质/状态 Hard Gate、Fit、Evidence、候选池及官方库覆盖缺口后，必须暂停等待采购员确认初版邀标供方。

### Phase B — 邀标四件套

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
04 {{项目名称}}-供方信息长名单.xlsx
```

- 01：单一 EML，供方邮箱仅放 BCC，不自动发送；正文不得残留 Markdown 标记；
- 02、03：实际对外附件；
- 03：必须复制企业原版模板；`N3` 保证金 = `CEILING(采购清单预估总金额 × 1%, 1000)`；
- 04：采购内部文件，只列人工确认初版供方及其已确认联系人/电话/邮箱，不外发。

## supplier-shortlist v0.5.0

### Phase A — 供方回复与短名单草稿

使用企业 `供方短名单模板.xlsx` 汇总供方实际回复，AI仅输出 `建议入围 / 待澄清 / 不建议入围`，随后必须暂停等待采购员确认。

### Phase B — 短名单报批

采购员确认后：

1. 固化最终 `{{项目名称}}-供方短名单.xlsx`；
2. 生成 `{{项目名称}}-短名单报批邮件.md`；
3. 生成 `{{项目名称}}-shortlist-handoff.yaml`；
4. 下游进入 `sourcing-strategy`。

Phase B 不允许重新评分、重新排序或改变采购员确认结果。

## sourcing-strategy v0.7.3

直接消费 confirmed requirement、官方库/保证金结果、人工确认 shortlist_handoff、历史采购分析和当前项目规则，输出：

```text
{{项目名称}}-strategy-data.yaml
{{项目名称}}-采购方案报告.docx
```

### 支出分析最低深度

有历史数据时必须尽可能形成：

- 整体实际支出 + 年化/预测支出；
- 配送区域金额/占比/订单数；
- 业务单元/军种/业态/项目金额与集中度；
- 订单金额段与订单碎片化；
- Top 5 SKU、Pareto 80%结构、数量/均价/金额/占比；
- 历史供应商集中度；
- 可比 SKU 价格分析；
- “免运额度/最低配送金额”的订单及履约依据；
- 至少3条数字化关键发现；
- 至少2条采购含义。

每个可用维度优先使用“数据表 → 结构特征 → 影响 → 采购动作”的写法。没有足够数据时降级分析并明确缺口，不得只写“暂无数据”。

### 当地供应市场行情最低深度

必须组合：

1. 内部供应市场信号：官方库供方数量、区域覆盖、邀约、回复、短名单、商务条件接受度；
2. 外部公开市场：供应格局、成本驱动、价格趋势、政策/标准；
3. 本地履约经济性：配送半径、区县/跨区域、小额高频订单、仓储、搬运、紧急配送、最低起送反馈；
4. 采购策略影响：至少3条与本项目直接相关的动作。

Web 可用且地区/品类明确时，应主动研究公开市场；`complete` 原则上至少3个独立来源并优先近12个月资料。具体成本比例仅在有可靠来源时允许填写。

### 内容质量 Gate

- 不得只填模板标题；
- 不得残留 `X / XX / XXX / X%`；
- 不得继承参考案例具体金额、供应商、日期、人员、成本比例或定标规则；
- 支出表格后必须有分析判断和采购含义；
- 市场章节必须有供应格局、成本驱动、价格/履约因素和采购动作；
- 免运/最低配送阈值属于 Human Decision，AI只能提出有依据的建议；
- 短名单先说明官方库→邀约→回复→入围漏斗，再逐家写可追溯理由；
- 风险优先按“触发场景→影响→处置”写；
- `strategy-data.yaml` 使用 schema `0.7.3` 记录 dimension coverage、order amount distribution、free shipping analysis、market cost structure 和 content depth checks；
- 分析深度不足不得标记 `ready_for_approval`。

详细规则见：

- `references/report-content-depth-rules.md`
- `references/report-content-quality-guide.md`
- `references/template-v3-content-blueprint.md`
- `references/procurement-plan-template-mapping.md`

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
15. sourcing-strategy 核心分析章节必须形成“事实 → 分析 → 判断 → 采购动作”。
16. 所有重要结论、统计和计算必须可追溯。

## v0.9.1

- 强化 `sourcing-strategy` 报告内容深度，升级至 v0.7.3；
- 支出分析覆盖整体、区域、业务单元/军种/业态、订单金额段、SKU/Pareto、历史供方、价格和免运/最低配送依据；
- 当地供应市场行情升级为内部供应信号 + 外部公开研究 + 成本/价格 + 本地履约 + 采购动作；
- `schemas/sourcing-strategy.schema.yaml` 升级为 0.7.3，增加多维支出、订单金额分布、免运分析、成本构成和 content depth checks；
- `historical-procurement-analysis` 升级至 v0.5.6，向策略报告提供更丰富的支出 Handoff；
- 新增/强化 `report-content-quality-guide.md`、`template-v3-content-blueprint.md`、`report-content-depth-rules.md`；
- 强化 `thin-analysis-regression`，防止报告退化为标题/占位式输出；
- 参考企业优秀案例只抽象写作结构，不提交真实项目数据。

## v0.9.0

- 将 `shortlist-approval` 合并进 `supplier-shortlist`；
- `supplier-shortlist` 升级至 v0.5.0，拆分 Phase A / Phase B；
- 删除独立 `shortlist-approval` Skill 入口；
- `sourcing-strategy` 只消费人工确认 shortlist_handoff；
- 主流程压缩为5个 Skill。
