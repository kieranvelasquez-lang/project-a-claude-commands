---
name: Slack message formatting rules
description: Correct Slack mrkdwn formatting to use when composing or sending Slack messages
type: feedback
---

When writing Slack messages (e.g. via slack_send_message), use these formatting rules:

- **Bold**: always `**double asterisk**` — never single `*asterisk*`
- **Italic**: `_underscore_`
- **Bold italic**: `**_combined_**`
- **Links**: always `<https://url|display text>` — never markdown `[text](url)`
- **Ampersands**: always raw `&` — never `&amp;`
- **Blank lines**: always one blank line between thematic sections
- **Message length**: Slack has a ~4000 character practical limit per message. If a post exceeds this, split into Part 1 / Part 2, repeating the title and marking the second _(continued)_.
- **No horizontal rules**: never use `---` in Slack messages — it causes an `invalid_blocks` error in the Slack MCP tool.

**Why:** User corrected this explicitly — these are the actual Slack mrkdwn rules; single asterisk and HTML entities don't render correctly.

**How to apply:** Any time composing a Slack message body for `slack_send_message` or similar tools.
