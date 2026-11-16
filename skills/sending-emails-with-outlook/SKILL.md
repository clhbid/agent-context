---
name: sending-emails-with-outlook
description: "Write an Outlook email body through the Microsoft 365 connector: HTML, paragraph spacing for a new message or a reply, and changing a draft. Use when drafting, replying to or sending email in Outlook."
---

# Sending emails with Outlook

How to write the body of any email made with the Microsoft 365 connector's mail tools. What to
say, and to whom, belongs to the task that asked for it. Find the tools by capability (create
draft, reply-all draft, delete draft, send), since their names change between connector versions.

**Leave a draft for the user to send** unless the task says to send it.

## Before drafting

- Settle every open choice with the user first, so the draft is sent as written.
- Show the text in chat as plain text, with a blank line between paragraphs and tables as tables,
  so the user can read it there or paste it into Outlook.
- End with the user's sign-off, taken from their recent sent mail.

## HTML

Set `bodyType` to `html`. The connector's allowlist rejects attributes, so write bare `<p>`, `<ul>`,
`<ol>`, `<li>`, `<b>` and `<table>` tags.

## Paragraph spacing

Outlook restyles every `<p>` when it saves the message, and how depends on whether the message
starts a thread:

| Message            | Outlook gives each `<p>` | Write                                                                                     |
| ------------------ | ------------------------ | ----------------------------------------------------------------------------------------- |
| New                | a 1em margin             | One `<p>` per paragraph, with nothing between them. A spacer doubles the gap              |
| Reply or reply-all | no margin                | `<p>&nbsp;</p>` between paragraphs, before and after each table, and before the sign-off |

Lists keep their own margin in both, so they need no spacer.

## Changing a draft

Update a new message's draft in place. To change a reply or reply-all draft, create a new one and
delete the old: updating its body replaces the quoted thread too.

## Without the connector

When the session has no claude.ai connectors, run a CLI session allowed only the draft tool:

```bash
claude -p "<task>" --allowedTools ToolSearch,mcp__claude_ai_Microsoft_365__outlook_create_draft
```
