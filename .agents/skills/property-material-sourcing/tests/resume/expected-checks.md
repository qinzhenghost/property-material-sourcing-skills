# Resume flow expected checks

- 用户提供最终需求清单时，不要求重跑 Phase 0/1，先校验该清单是否为当前项目最新确认版本。
- 已完成 Phase 2 邀标四件套但没有供方实际回复时，状态为 waiting_supplier_replies。
- 用户后续提供真实供方回复后，从 Phase 3A 继续。
- 已人工确认最终短名单且存在有效 shortlist-handoff 时，可直接进入 Phase 4。
- 用户说“继续”时从 last_verified_checkpoint 恢复，不重复无变化的已确认阶段。
- 上游需求、初版供方或最终短名单发生实质变化时，受影响下游产物必须标记 superseded，并列出需要重跑的阶段。
