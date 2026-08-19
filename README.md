# Property Material Sourcing Skills — v0.11.0

物业物资类 AI 邀标寻源 Skill 库。

## 唯一运行目录

从 v0.11.0 起，仓库只保留一个 Skill Source of Truth：

```text
.agents/skills/
```

端到端任务推荐只调用统一主 Skill：

```text
.agents/skills/property-material-sourcing/
```

主 Skill 负责识别项目当前状态、调用内部专业模块、维护 Human Gate、复用最新确认产物并从断点继续。

根目录旧 `skills/` 兼容镜像已删除，不再维护双目录。

## 统一流程

```text
property-material-sourcing
        ↓
Phase 0  historical-procurement-analysis（可选）
        ↓
Phase 1  material-requirement-analysis
        ↓
Phase 2  sourcing-invitation
           ├─ 官方供方匹配
           ├─【采购员确认初版邀标供方】
           └─ 邀标四件套
        ↓
waiting_supplier_replies
        ↓
Phase 3  supplier-shortlist
           ├─ 实际回复分析
           ├─ AI短名单建议
           ├─【采购员确认最终短名单】
           └─ 短名单报批 + shortlist-handoff
        ↓
Phase 4  sourcing-strategy
           ├─ V3深度支出分析
           ├─ 当地供应市场行情
           └─ 采购方案报告.docx
```

## 内部专业模块

| 模块 | 当前版本 | 职责 |
|---|---:|---|
| `historical-procurement-analysis` | 0.5.6 | 协议选择、有效订单过滤、12个月年化、年度增量、多维支出 Handoff |
| `material-requirement-analysis` | 0.1.2 | P0/P1/P2 需求诊断、澄清/最终需求清单 |
| `sourcing-invitation` | 0.4.x | 官方供方匹配、人工确认、邀标四件套 |
| `supplier-shortlist` | 0.5.0 | 供方回复、AI建议、人工最终短名单、报批 Handoff |
| `sourcing-strategy` | 0.7.4 | V3 支出/市场分析、采购方案报告 |

这些模块只保留在 `.agents/skills/` 中，作为规则、模板和测试的权威实现。用户无需手工串联。

## 统一状态机

主 Skill 可维护：

```text
{{项目名称}}-procurement-workflow-state.yaml
```

Schema：

```text
schemas/property-material-sourcing-workflow.schema.yaml
```

状态包括：

- `phase_0_history_optional`
- `phase_1_requirement`
- `phase_2_supplier_sourcing`
- `waiting_supplier_replies`
- `phase_3_shortlist`
- `phase_4_strategy`
- `completed`
- `blocked`

用户说“继续”时，应从最近一个已验证 checkpoint 接着执行，不重复已经人工确认且输入未变化的阶段。

## 关键 Human Gates

整合后仍禁止自动越过：

1. 历史协议范围确认；
2. 必要时年度增量确认；
3. P0 需求澄清；
4. 初版邀标供方确认；
5. 邀标邮件/附件人工发送；
6. 最终短名单确认；
7. 预算、定标方式、中标家数、份额、目标降本率、评标人员等正式采购决策；
8. 最终采购方案报批。

## Phase 2 固定交付物

采购员确认初版供方后生成：

```text
01 {{项目名称}}-邀标邮件.eml
02 {{项目名称}}-最终需求清单.xlsx
03 {{项目名称}}-招标意向征集登记表.xlsx
04 {{项目名称}}-供方信息长名单.xlsx
```

- EML：每项目只生成一个，供方邮箱仅 Bcc，不自动发送，不嵌入附件，正文不得残留 Markdown；
- 02、03：对外附件；
- 03：必须复制企业原模板；N3 投标保证金按当前项目可追溯预估金额 × 1%，向上取整至 1000 元整数倍；
- 04：仅采购内部使用，展示供方名称、联系人、电话、邮箱、信息来源、状态、备注，不展示供方编码。

## Phase 3 固定逻辑

AI 只可给：`建议入围 / 待澄清 / 不建议入围`。

采购员确认最终短名单前不得进入正式报批；确认后不得重新评分、排序、增加或替换供方。

## Phase 4 V3 内容深度

`sourcing-strategy` v0.7.4 必须按企业 V3 模板生成，而不是只填标题。

### 支出分析

必须完成模板四个核心子块：

1. 整体；
2. 军种/业务单元/项目/业态（按真实字段映射）；
3. SKU；
4. 免运额度/最低配送金额。

并优先补充配送区域、订单金额段、历史供方、价格等可用维度。

有充分历史数据时至少形成数字事实、数字表、关键发现和采购动作；数据不足时按降级规则明确缺口，不能只写“暂无数据”。

### 当地供应市场行情

至少覆盖：

- 内部供应市场信号；
- 当地供应格局；
- 成本构成；
- 价格/市场风险趋势；
- 本地仓储、运输、项目配送等履约因素；
- 对本次采购的具体动作。

Web 可用且地区/品类明确时应主动研究公开市场。公网市场资料只用于行情分析，绝不能新增官方邀标候选供方。

## 模板保护

统一主 Skill 不搬迁已经验证的二进制模板，仍由内部模块按 `.agents/skills/` 下的原路径使用：

- 历史采购分析需求清单模板；
- 标准需求清单模板；
- 招标意向征集登记表模板；
- 供方信息长名单模板；
- 供方短名单模板；
- 企业《物资采购方案报告模板.docx》。

## 核心 Guardrails

1. V1 只做物业物资类采购。
2. 候选供方身份只能来自企业内部官方供方库。
3. 公网资料不得新增邀标候选供方。
4. AI 推荐不能替代采购员确认。
5. 所有邮件只生成草稿，不自动发送。
6. 不编造联系人、邮箱、电话、价格、资质、业绩、预算、制度、审批人。
7. 不把模板示例或历史案例数据当当前项目事实。
8. 不混用含税/未税，不把部分周期金额当全年金额。
9. 下游优先消费最新 `human_confirmed / final` 产物，旧草稿标记为 `superseded`。
10. 所有重要结论、统计和计算必须可追溯。

## v0.11.0

- 删除根目录 `skills/` 兼容镜像；
- `.agents/skills/` 成为唯一 Skill Source of Truth；
- README / AGENTS 统一移除双目录维护说明；
- 检查发现 GitHub 中 `供方信息长名单模板.xlsx` ZIP 结构损坏，已用验证通过的 8 列无“供方编码”版本替换；
- 供方长名单继续固定为：序号、供方名称、联系人姓名、联系电话、邮箱地址、联系信息来源、联系信息状态、备注。
