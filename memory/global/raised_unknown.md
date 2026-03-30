---
name: Omit raised field when unknown
description: Do not include "Raised: Unknown" in dealflow posts — omit the field entirely if no funding info is available
type: feedback
---

Never include `| _Raised: Unknown_` in Net New post entries. If no funding information is findable, simply omit the raised field entirely.

**Why:** It adds no value to the reader.

**How to apply:** During Step 6b enrichment, if WebSearch returns no funding info, leave the `| _Raised:` field out of that entry entirely.
