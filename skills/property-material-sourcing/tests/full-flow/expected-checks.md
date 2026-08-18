# Unified workflow expected checks

- 主入口为 property-material-sourcing。
- 有历史数据时从 Phase 0 开始；无历史数据时允许跳过 Phase 0。
- 协议范围、P0、初版邀标供方、最终短名单等人工确认节点不得自动跳过。
- 候选供方只来自企业官方供方库。
- 邀标阶段交付单一 BCC EML、最终需求清单、招标意向征集登记表、内部供方信息长名单。
- EML 正文不得包含 Markdown 标记。
- 内部供方信息长名单不得展示供方编码。
- 未收到实际供方回复时状态为 waiting_supplier_replies。
- AI 短名单建议不得直接成为最终短名单。
- 采购策略报告必须使用企业模板并通过 sourcing-strategy 的 V3 支出分析和当地供应市场行情内容深度检查。
- 用户说继续时从最近已验证 checkpoint 恢复，不重复无变化的已确认阶段。
