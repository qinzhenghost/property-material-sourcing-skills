# EML Markdown-Free Regression Check

This file defines a regression check for the final `.eml` deliverable.

The source template may be stored as Markdown, but the decoded EML body must be plain text or HTML and must not contain Markdown formatting markers.

Fail if the decoded body contains Markdown formatting syntax such as:

- ATX heading markers at line start (`# `, `## `, etc.)
- bold or emphasis markers (`**`, `__`)
- fenced or inline code markers (backticks)
- Markdown links (`[text](url)`)
- blockquote markers at line start (`> `)
- unordered Markdown list markers at line start (`- `, `* `, `+ `)

Plain-text numbered sentences and the Unicode bullet `•` are allowed.

Expected failure state:

`BLOCK/HOLD: eml_markdown_marker_detected`
