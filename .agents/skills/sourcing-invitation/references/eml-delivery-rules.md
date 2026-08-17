# 单一 BCC EML 交付规则

## 目标

`sourcing-invitation` 只生成一封 `.eml` 草稿，所有人工确认且邮箱已确认的供方统一放入 `Bcc`。

## 固定交付物

```text
{{项目名称}}-邀标邮件.eml
{{项目名称}}-最终需求清单.xlsx
{{项目名称}}-招标意向征集登记表.xlsx
```

三者均为独立文件。

## 收件人规则

- `Bcc`：已确认参与邀标且邮箱来源可追溯的供方邮箱；
- `To`：不得填写任何供方邮箱；仅可填写采购方明确提供的本方发送/归档邮箱；
- `Cc`：仅可填写采购员明确确认的内部抄送邮箱；
- `From`：仅在用户/运行环境明确提供时填写；
- 缺失或冲突的供方邮箱进入 `missing_recipient_emails` / `needs_email_confirmation`，不得猜测。

## 附件规则

EML 中：

```text
attachments_embedded = false
```

即：

- 不把 Excel 编码到 MIME 附件；
- 正文只引用两个附件文件名；
- 两个 Excel 独立交付；
- 采购员实际发送前人工添加附件。

## 状态

- `ready_for_human_review`：所有已选供方邮箱均已确认；
- `draft_missing_recipient_email`：至少一个已选供方缺邮箱；
- `draft_recipient_email_conflict`：至少一个邮箱来源存在冲突。

无论哪种状态都不得自动发送。

## 隐私与防误发

- 多家供方不得放入 To/Cc；
- 供方之间不得看到其他受邀供方邮箱；
- 不为每家供方自动生成单独 EML；
- BCC 只能来自人工确认邀标名单；
- 被排除、hold、待补证供方不得进入 BCC。
