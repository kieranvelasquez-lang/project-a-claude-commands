---
name: Morning Recap Affinity list format
description: Affinity pre-scraping list in morning-recap must use bullet points, one entry per line
type: feedback
---

Use `•` bullet points for every entry in the Affinity check list posted after the Morning Recap. One entry per line.

Format:
```
• CompanyName — https://url.com
• FullName (LinkedIn) — https://linkedin.com/in/handle
```

Use plain URLs (not Slack `<url|text>` syntax).

**Why:** Without bullet points, Slack collapses all entries into a single paragraph and URLs bleed into the next entry's name (e.g. Kevin Holmes's LinkedIn link merged into Philip Kwok's name). Bullets force each entry onto its own line.

**How to apply:** In Step 7 of `morning-recap.md` and in the remote trigger prompt, the Affinity list must always use `•` prefix per entry.
