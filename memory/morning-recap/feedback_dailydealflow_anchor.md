---
name: daily-dealflow anchor timestamp
description: When running /daily-dealflow, only pull Slack messages posted after the most recent Daily Dealflow post sent by Kieran Velasquez
type: feedback
---

When running the `/daily-dealflow` skill, use the `oldest` parameter on `slack_read_channel` to fetch only messages posted after the most recent Daily Dealflow summary message sent by Kieran Velasquez in #deal-flow. Do not download the full channel history.

**Why:** Pulling 100 messages without a timestamp filter returns the entire channel history, which exceeds token limits and wastes time.

**How to apply:** In Step 3, find the timestamp of the last Daily Dealflow post (sent by Kieran Velasquez, starts with `**Daily Dealflow —`) — either from context the user provides or by fetching a small sample of recent messages first. Then use that timestamp as `oldest` in the main fetch.
