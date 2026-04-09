---
description: Screen an ad-hoc deal flow list pasted by the user — enrich entries where possible, route to theses, and post to #automation-tests
allowed-tools: Read, WebSearch, WebFetch, mcp__claude_ai_Slack__slack_send_message, mcp__claude_ai_Slack__slack_search_users
---

# Ad-hoc Deal Flow Screen

## Team member → thesis mapping

| Thesis | Members |
|---|---|
| Future of Autonomous Work | Daria Gneusheva, Omar Hedeya |
| Fintech | Malin Posern, Marjorie Lengereau |
| Global Supply Chain | Philipp Werner, Oskar Lingk |
| European Resilience | Jack Wang, Miha Pavlovic |
| Surf and Turf | Ciara Gumsheimer |

---

## Step 1 — Collect the list

If the user has not already pasted the list in their prompt, ask:

> "Paste the deal flow list — one entry per line. Include any URLs, LinkedIn links, or descriptions if you have them."

Accept any format: bare names, names with URLs, LinkedIn profile links, company names with descriptions, mixed formats. Do not require structure.

---

## Step 2 — Parse each entry

The list may contain section headers like "Pipeline", "Declined Q1", "Portfolio - Closed", "Market Radar", or similar. **Ignore these labels entirely** — process every company and person entry the same way regardless of which section it appears in. Never skip entries based on a section label.

For each item in the list, extract:

- **Name** — company or person name
- **URL** — company website or LinkedIn profile URL, if provided
- **Description** — any inline text following the name/URL (keep verbatim — use this even if no URL is found during enrichment)
- **Type** — `company` or `person` (LinkedIn)

---

## Step 3 — Enrich entries

For each entry, follow this enrichment path:

### URL was provided

- **Company URL**: WebFetch the homepage. Extract a one-sentence description of what the company does. If WebFetch fails, fall back to WebSearch with the company name + "startup".
- **LinkedIn URL**: LinkedIn blocks automated fetching — skip enrichment. Name only.

### No URL provided

- **Company name only**: Run WebSearch for "[company name] startup" or "[company name] what does it do". Only use a result if the company name in the result is an **exact match** to the name in the list — no partial matches, no assumed synonyms, no "this might be the same as" logic.
  - Exact match found → WebFetch the company homepage for a one-sentence description.
  - No exact match found → **flag the entry**.
- **Person name only (no LinkedIn URL)** → **flag the entry** — do not guess which profile it is.

### Flagged entries — pause before composing

After processing all entries, if any are flagged (no URL found, ambiguous match), stop and ask:

> "I couldn't find exact matches for the following. Provide a URL for each, or type 'skip' to leave them unlinked:
> - [Entry 1]
> - [Entry 2]"

Accept user-provided URLs and re-run enrichment for those entries. If user says 'skip' for an entry, include the name only — no URL, no description — in the output. Only proceed to Step 4 once all flags are resolved.

---

## Step 4 — Route each entry

First, load staged corrections: read `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md`. Any company listed there overrides default routing.

**Default routing table:**

| What they build | Thesis |
|---|---|
| AI agents, orchestration, LLM infrastructure, dev tools, enterprise AI-native SaaS, robotics | Future of Autonomous Work |
| Fintech, payments, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | Fintech |
| Supply chain, logistics, manufacturing, materials | Global Supply Chain |
| Defense, hardware, chips, non-GNSS navigation, industrial security, cloud infrastructure | European Resilience |
| Health, biotech, edtech, consumer, gaming, fitness, energy, creator, construction, agriculture | Surf and Turf |

**Critical routing test:** Is this company *building* AI, or *using* AI for a specific domain?
- Building AI tools / infrastructure / agents → Future of Autonomous Work
- Using AI to solve a domain problem (e.g. AI for insurance, AI for logistics) → route to that domain's thesis

Never default to Future of Autonomous Work without applying this test first.

**Hardcoded routing rules:**
- Cybersecurity (pentesting, infosec, security tooling) → Future of Autonomous Work, not European Resilience
- AI sales tools (commissions, sales enablement, revenue ops) → Surf and Turf, not Fintech
- Blockchain / crypto / web3 → Fintech, not Future of Autonomous Work
- Energy companies → Surf and Turf by default. If software-based → add `| _Action: Oskar Lingk_`. If hardware-based → add `| _Action: Miha Pavlovic_`. (Resolve to `<@USERID>` in Step 6.)

If a company still has no description after enrichment + user input and cannot be routed with confidence, place it in **⚠️ Flagged for Review**.

---

## Step 5 — Resolve @mention user IDs

Use `slack_search_users` to look up the Slack user ID for each team member whose thesis section will appear in the post. Format as `<@USERID>`.

---

## Step 6 — Compose the post

Use this format exactly:

```
**Ad-hoc Deal Flow Screen — [Month D, YYYY]**

**Future of Autonomous Work** <@DARIA_ID> <@OMAR_ID>
- <https://company.com|CompanyName> — One-sentence description.

**Fintech** <@MALIN_ID> <@MARJORIE_ID>
- <https://company.com|CompanyName> — One-sentence description.

**Global Supply Chain** <@PHILIPP_ID> <@OSKAR_ID>
- <https://company.com|CompanyName> — One-sentence description.

**European Resilience** <@JACK_ID> <@MIHA_ID>
- <https://company.com|CompanyName> — One-sentence description.

**Surf and Turf** <@CIARA_ID>
- <https://company.com|CompanyName> — One-sentence description.

---

**⚠️ Flagged for Review**
- CompanyName — reason
```

**Section order (fixed — never deviate):**
1. Future of Autonomous Work
2. Fintech
3. Global Supply Chain
4. European Resilience
5. Surf and Turf

Only include sections that have entries. Only include Flagged for Review if there are flags.

**Formatting rules:**
- Bold: `**double asterisk**`
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never `[text](url)`
- @mentions: `<@USERID>` — never plain `@name`
- Ampersands: raw `&` — never `&amp;`
- One blank line between thesis sections
- Do not include `| _Raised:_` — no funding info in this post
- Do not append `_Sent using Claude_`
- LinkedIn profiles with no description: show link and name only — no description fragment
- Entries with no URL after all enrichment attempts + user input: show plain name only, no link, no description
- Action item format (energy routing): `| _Action: <@USERID>_` appended at end of line
- If post exceeds ~4000 characters, split into Part 1 / Part 2 with title repeated and _(continued)_ on the second

---

## Step 7 — Post to #automation-tests

Post to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then output to terminal:
> "Screened [N] entries → posted to #automation-tests."

---

## Step 8 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 9 — Learn from corrections (only if user provides them)

For each routing correction:

1. Determine if **company-specific** or a **general rule**.
   - General rules → ask Kieran to confirm before editing the skill directly.
   - Company-specific → stage in corrections file.

2. Append to `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md`:
```
- CompanyName → Thesis Name | added YYYY-MM-DD
```

3. Tell the user:
> "Learned [N] new correction(s): [list]. Staged for future runs."
