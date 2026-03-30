---
description: Generate the Net New to Affinity post from Affinity scraper results and post to #automation-tests
allowed-tools: Read, Write, WebSearch, WebFetch, mcp__claude_ai_Slack__slack_send_message, mcp__claude_ai_Slack__slack_search_users
---

# Net New to Affinity

Run this after `/daily-dealflow` has posted the Summary and raw Affinity list to #automation-tests, and after you have run your Affinity scraper against that list.

## Team member Slack display names

| Thesis | Members |
|---|---|
| Future of Autonomous Work | Daria Gneusheva, Omar Hedeya |
| Fintech | Malin Posern, Marjorie Lengereau |
| European Resilience | Jack Wang, Miha Pavlovic |
| Global Supply Chain | Philipp Werner, Oskar Lingk |
| Surf and Turf | Ciara Gumsheimer |

---

## Step 1 — Load corrections

Read `~/.claude/projects/-Users-kvelasquez-Projects/memory/daily-dealflow-corrections.md`. Load any staged corrections as overrides for thesis routing. Note briefly if any exist.

---

## Step 2 — Accept Affinity scraper results

Ask:
> "Paste your Affinity scraper results (found / not-found list)."

Wait for the user's response before proceeding.

---

## Step 3 — Confirm Net New entries

**Critical:** The scraper has a high false-negative rate. Only entries the user's manual check explicitly confirms as "not in Affinity" proceed to enrichment and the Net New post. Do not rely on scraper output alone.

Separate the list into:
- **Confirmed Net New** — user explicitly says not in Affinity
- **Found / uncertain** — skip these; do not include in Net New post

---

## Step 4 — LinkedIn double-check

For any LinkedIn profiles in the confirmed Net New list, output clickable URLs so the user can open them directly:

> "Here are the Net New LinkedIn profiles — click through and paste a one-liner per person:
> • https://www.linkedin.com/in/slug-one/
> • https://www.linkedin.com/in/slug-two/
>
> e.g. `pierre-dorge → Co-founder Fileforge, building in stealth`"

Use whatever the user provides as the profile description — do not attempt WebFetch on LinkedIn URLs.

---

## Step 5 — Enrich Net New companies

For each confirmed Net New **company** (not LinkedIn profiles):

### Verify description via website
Use WebFetch on the company's actual website homepage. Always verify or replace the Slack description with what the company actually says about itself. If the site is down or returns an error, try WebSearch (company name + "startup" or "what does it do") before giving up.

If both fail: flag the entry — do not invent a description.

### Research funding
Use WebSearch to find funding info (search: company name + "funding" or "raises" + site:crunchbase.com or site:techcrunch.com or press releases).

Include whatever is findable — amount, round type, lead investor, date. Format flexibly based on what's available:
- `_Raised: $1.5M, Pre-Seed_`
- `_Raised: $1.52M, Seed, EWOR_`
- `_Raised: Pre-Seed_`
- `_Raised: £4–5M Seed, raising Jun 2026_`

If no funding info is findable, **omit the Raised field entirely** — never write `_Raised: Unknown_`.

Do not include approximate amounts without a `~` prefix.
Do not include months/years unless confirmed.
LinkedIn profiles do not get a Raised field.

### Flag when needed
Flag only when genuinely needed:
- Broken or dead links
- Contradictory funding info
- Site down and WebSearch also returned nothing
- Truly unresolvable routing with zero public info and no Slack context

Do not flag for ambiguous routing — make a best-guess using founder background, Slack context, or who it was tagged to.
Never skip entries.

---

## Step 6 — Resolve @mention user IDs

Use `slack_search_users` to look up the Slack user ID for each team member whose thesis section appears in the Net New output. Format pings as `<@USERID>`.

---

## Step 7 — Compose Net New to Affinity post

Match this format exactly:

```
**Net New to Affinity — [Month D, YYYY]**
_Entries not yet tracked in Affinity · [Month D, YYYY]_

**Future of Autonomous Work** <@DARIA_ID> <@OMAR_ID>
- <https://company.com|CompanyName> — One-sentence description. | _Raised: $XM, Round Type_
- <https://linkedin.com/in/handle|Full Name> — One-sentence bio/context.

**Global Supply Chain** <@PHILIPP_ID> <@OSKAR_ID>
- <https://company.com|CompanyName> — One-sentence description.

**Fintech** <@MALIN_ID> <@MARJORIE_ID>
- <https://company.com|CompanyName> — One-sentence description. | _Raised: Pre-Seed_

**Surf and Turf** <@CIARA_ID>
- <https://company.com|CompanyName> — One-sentence description. | _Raised: $XM, Seed, Lead Investor_

**European Resilience** <@JACK_ID> <@MIHA_ID>
- <https://company.com|CompanyName> — One-sentence description. | _Raised: Pre-Seed_

---

**⚠️ Flagged for Review**
- CompanyName — reason
```

Rules:
- Only include thesis sections that have entries
- Only include "Flagged for Review" if there are actual flags
- One blank line between each thesis section
- No lead investor unless confirmed
- Omit `| _Raised:_` entirely if no funding info is available — never write `_Raised: Unknown_`
- Do NOT include `_Sent using Claude_`
- If post exceeds ~4000 characters, split into Part 1 / Part 2

### Formatting rules (CRITICAL — never deviate)

- Bold: `**double asterisk**`
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never markdown `[text](url)` format
- @mentions: `<@USERID>` — never plain `@name`
- Ampersands: raw `&` — never `&amp;`
- Descriptions end with `.` before ` | _Raised:_`
- No internal commentary in entries
- No "sourced by" attribution
- Do NOT append `_Sent using Claude_`

---

## Step 8 — Post to #automation-tests

Post to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then output:
> "Net New post is live in #automation-tests. Copy it to #deal-flow when ready."

---

## Step 9 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 10 — Learn from corrections (only if user provides them)

For each routing correction provided:

1. Determine if this is **company-specific** or **general**.
   - **General rules** → ask Kieran to confirm before baking into either skill's hardcoded routing.
   - **Company-specific corrections** → staging file.

2. Check if company already exists in `~/.claude/projects/-Users-kvelasquez-Projects/memory/daily-dealflow-corrections.md`. If file doesn't exist, create it with:

```markdown
# Daily Dealflow — Corrections Memory

## Staged Corrections (not yet baked into skill)
<!-- Format: CompanyName → Thesis | added YYYY-MM-DD -->
```

3. Append only new entries:
```
- CompanyName → Thesis Name | added YYYY-MM-DD
```

4. Tell the user:
> "Learned [N] new correction(s): [list]. Staged for future runs. Let me know if any should be baked into the skill as a permanent rule."
