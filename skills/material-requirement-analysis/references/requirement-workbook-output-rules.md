# 需求清单 Excel 强制输出规则

当 `material-requirement-analysis` 人工澄清后 `P0 blocker count = 0`，必须生成需求清单 Excel。P1/P2 尚未解决不构成不生成 Excel 的理由。

- P0 = 0 且仍有 P1/P2：输出 `{{项目名称}}-澄清版需求清单.xlsx`。
- P0 = 0 且关键需求和商务条件已确认：输出 `{{项目名称}}-最终需求清单.xlsx`。

模板优先级：用户原 Excel副本 > 当前企业模板 > `templates/标准需求清单模板.xlsx`。不得覆盖原文件。

采购需求字段至少包含序号、区域、名称、品牌（如有）、规格、单位、起订量（如有）、年度预计采购量、备注及可映射的项目级商务条件。

含税单价、未税单价、税率、含税总价属于供方报价字段，保留但必须为空，不得写历史价、旧报价、市场价或AI测算价。

未解决 P1/P2 应在备注或“待确认事项”中留痕。

输出前必须核验 Excel 与 Structured Requirement / Handoff 的 SKU、区域、品牌、规格、数量和商务条件一致。

P0 清零后若只有文字/YAML/JSON而无 Excel，则本 Skill 执行状态为 `incomplete`。
