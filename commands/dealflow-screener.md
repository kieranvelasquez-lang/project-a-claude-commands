---
description: Screen an ad-hoc deal flow list — enriches and routes companies (Part 1), then routes LinkedIn profiles with a user context step (Part 2), posting both to #automation-tests
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

Accept any format. Do not require structure.

---

## Step 2 — Parse and separate entries

The list may contain section headers (e.g. "Pipeline", "Declined Q1", "Portfolio - Closed", "Market Radar", "Tracking"). **Ignore these labels entirely** — process every entry the same way regardless of which section it appears in. Never skip entries based on section labels.

Separate all entries into two buckets:

**Companies** — has a company website URL, or is clearly a company/product name (not a person's name).

**Profiles** — has a LinkedIn URL, or is clearly a person's name (with or without a title/background note).

For each entry in both buckets, capture:
- **Name**
- **URL** (company website or LinkedIn URL, if provided)
- **Description/context** — any inline text following the name or URL (keep verbatim)

---

## Step 3 — Enrich companies

For each company entry:

### URL was provided
- **Company URL**: WebFetch the homepage for a one-sentence description. If WebFetch fails, try WebSearch with company name + "startup".
- If the original list included an inline description, use that even if enrichment fails.

### No URL provided
- Run WebSearch for "[company name] startup". Only use a result if the company name is an **exact match** — no partial matches, no assumed synonyms.
  - Exact match found → WebFetch for a one-sentence description.
  - No exact match found → **flag the entry**.

### Flagged companies — pause before composing Part 1

After processing all companies, if any are flagged, stop and ask:

> "I couldn't find exact matches for the following companies. Provide a URL for each, or type 'skip' to leave them unlinked:
> - [Company 1]
> - [Company 2]"

If user provides URLs, re-run enrichment. If 'skip', include name only — no link, no description. Only proceed once all flags are resolved.

---

## Step 4 — Route companies

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
- Using AI to solve a domain problem → route to that domain's thesis

Never default to Future of Autonomous Work without applying this test first.

**Hardcoded routing rules:**
- Cybersecurity (pentesting, infosec, security tooling) → Future of Autonomous Work, not European Resilience
- AI sales tools (commissions, sales enablement, revenue ops) → Surf and Turf, not Fintech
- Blockchain / crypto / web3 → Fintech, not Future of Autonomous Work
- Energy companies → Surf and Turf by default. If software-based → add `| _Action: Oskar Lingk_`. If hardware-based → add `| _Action: Miha Pavlovic_`. (Resolve to `<@USERID>` in Step 7.)

If a company still cannot be routed with confidence, place it in **⚠️ Flagged for Review**.

---

## Step 5 — Handle profiles

Sort all profile entries into two groups:

**Has context** — the entry includes an inline description, background note, or enough information to route confidently (e.g. "Lars Rehfeldt — Aegiron", "Andrei Ciobotar — background in AI/IoT"). Route these directly using the same thesis routing table. The inline text becomes the description shown in Part 2.

**No context** — a LinkedIn URL or bare name with no inline description. Cannot route without more information.

If any profiles have no context, pause and output to the terminal:

> "Here are [N] profiles I couldn't route — no context available. Click through each and reply with a brief note (background, what they're working on, which thesis you think):
> 1. [Name — linkedin.com/in/slug] (or just [Name] if no URL)
> 2. ...

Wait for the user's response. Accept notes in any format (e.g. "1. robotics, ex-Palantir", "3. crypto, skip"). Once received, route all profiles using the notes provided. If the user provides a thesis directly, use it. If they say 'skip' for an entry, omit it from Part 2 entirely.

If all profiles already have context, skip this pause and proceed directly.

**Profile routing** uses the same thesis table as companies — route based on domain expertise and background:
- AI/ML engineers, AI researchers, AI infra → Future of Autonomous Work
- Fintech, crypto, compliance professionals → Fintech
- Supply chain, manufacturing, logistics → Global Supply Chain
- Defense, aerospace, hardware, robotics (defense context) → European Resilience
- Health, consumer, construction, energy, real estate → Surf and Turf

---

## Step 6 — Resolve @mention user IDs

Use `slack_search_users` to look up the Slack user ID for each team member whose thesis section appears in either part. Format as `<@USERID>`.

---

## Step 7 — Compose Part 1 (Companies)

```
**Ad-hoc Deal Flow Screen — [Month D, YYYY] (Part 1 of 2 — Companies)**

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

**⚠️ Flagged for Review**
- CompanyName — reason
```

**Section order (fixed — never deviate):**
1. Future of Autonomous Work
2. Fintech
3. Global Supply Chain
4. European Resilience
5. Surf and Turf

**Formatting rules:**
- Bold: `**double asterisk**`
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never `[text](url)`
- @mentions: `<@USERID>` — never plain `@name`
- Ampersands: raw `&` — never `&amp;`
- One blank line between thesis sections
- Do not include `| _Raised:_`
- Do not append `_Sent using Claude_`
- Entries with no URL: plain name only, no link; use description if provided in the original list
- Action item format: `| _Action: <@USERID>_` appended at end of line
- Only include sections that have entries; only include Flagged if there are flags
- If Part 1 exceeds ~4000 characters, split further: Part 1a / Part 1b with _(continued)_ on second

---

## Step 8 — Compose Part 2 (Profiles)

```
**Ad-hoc Deal Flow Screen — [Month D, YYYY] (Part 2 of 2 — Profiles)**

**Future of Autonomous Work** <@DARIA_ID> <@OMAR_ID>
- <https://linkedin.com/in/slug|Full Name> — background context note.

**Fintech** <@MALIN_ID> <@MARJORIE_ID>
- <https://linkedin.com/in/slug|Full Name> — background context note.

**Global Supply Chain** <@PHILIPP_ID> <@OSKAR_ID>
- <https://linkedin.com/in/slug|Full Name> — background context note.

**European Resilience** <@JACK_ID> <@MIHA_ID>
- <https://linkedin.com/in/slug|Full Name> — background context note.

**Surf and Turf** <@CIARA_ID>
- <https://linkedin.com/in/slug|Full Name> — background context note.
```

**Formatting rules:**
- Same section order and formatting rules as Part 1
- If a LinkedIn URL is available: `<https://linkedin.com/in/slug|Full Name>`
- If no LinkedIn URL: plain `Full Name` only
- Description is the background/context note from the original list or from the user's notes — keep concise, one line
- No `| _Raised:_`, no Flagged section (profiles that couldn't be routed are omitted)
- If Part 2 exceeds ~4000 characters, split further: Part 2a / Part 2b with _(continued)_ on second

---

## Step 9 — Post to #automation-tests

Post Part 1 first, then Part 2, to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then output to terminal:
> "Screened [N] companies (Part 1) + [M] profiles (Part 2) → posted to #automation-tests."

---

## Step 10 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 11 — Learn from corrections (only if user provides them)

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
