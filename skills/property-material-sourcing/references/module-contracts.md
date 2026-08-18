# Unified module contracts

`property-material-sourcing` 是唯一推荐入口；以下 5 个目录是内部专业模块。

| Phase | Internal module | Primary input | Human gate | Primary output |
|---|---|---|---|---|
| 0 | historical-procurement-analysis | 历史订单 | 协议范围；必要时年度增量 | 历史测算需求清单 + strategy_report_handoff |
| 1 | material-requirement-analysis | 原始/历史测算需求 | P0 澄清 | 澄清版/最终需求清单 |
| 2 | sourcing-invitation | 最终需求 + 官方供方库 | 初版邀标供方；人工发送 | EML + 2个外部Excel + 内部长名单 |
| 3 | supplier-shortlist | 实际供方回复 | 最终短名单 | 最终短名单 + 报批邮件 + shortlist-handoff |
| 4 | sourcing-strategy | 最终需求 + 可追溯供方结果 + 最终短名单 + 历史分析 | 预算/定标等正式决策；最终报批 | strategy-data.yaml + 采购方案报告.docx |

## Handoff rules

1. 下游只消费最新 `human_confirmed/final` 版本。
2. 上游输入发生实质变化时，受影响下游产物必须标记 `superseded`。
3. AI recommendation 不能升级成人工确认状态。
4. 公开市场研究只进入 Phase 4 market intelligence，不进入 Phase 2 candidate identity。
5. 二进制模板继续由各内部模块原路径管理，统一主 Skill 不复制、不重建模板。

## Resume mapping

- 只有历史订单：Phase 0。
- 已有历史测算/原始需求：Phase 1。
- 已有最终需求：Phase 2。
- 已生成邀标四件套但未收到回复：waiting_supplier_replies。
- 已有供方实际回复：Phase 3A。
- 已人工确认最终短名单：Phase 3B 或 Phase 4。
- 已有 confirmed shortlist-handoff：Phase 4。

无法确定状态时，优先检查实际产物和 Human Gate，不按文件名猜测确认状态。
