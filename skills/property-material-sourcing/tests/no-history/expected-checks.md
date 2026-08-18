# No-history flow expected checks

- 没有历史订单时 Phase 0 标记 skipped，不产生伪造历史数据。
- 直接进入 Phase 1 需求分析。
- P0 未解决不得进入 Phase 2。
- 最终需求确认后进入 Phase 2 官方供方匹配。
- 后续初版供方确认、等待实际回复、最终短名单确认等 Human Gate 与完整流程一致。
- Phase 4 没有历史支出时按 sourcing-strategy 的降级规则生成计划支出/数量结构，并明确数据限制，不得伪造历史金额。
