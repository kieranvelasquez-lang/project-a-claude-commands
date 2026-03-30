---
name: LinkedIn URL formatting in raw list
description: Always output full LinkedIn URLs (https://www.linkedin.com/in/slug/), never truncated slugs like /in/slug/
type: feedback
---

Always output full LinkedIn URLs when presenting the raw list for Affinity scraping or any other output — e.g. `https://www.linkedin.com/in/julianschwarzkopf/`, never `/in/julianschwarzkopf/`.

**Why:** Truncated slugs are unusable — the user can't click them, scraper can't hit them, and they look broken.

**How to apply:** Whenever outputting LinkedIn URLs anywhere in the daily-dealflow flow (raw list, enriched list, composed posts), always use the full `https://www.linkedin.com/in/` prefix.
