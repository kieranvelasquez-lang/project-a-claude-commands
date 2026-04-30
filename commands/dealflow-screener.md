---
description: Screen an ad-hoc deal flow list — enriches and routes companies and profiles into one unified list, posting to #automation-tests
allowed-tools: Read, WebSearch, WebFetch, mcp__claude_ai_Slack__slack_send_message, mcp__claude_ai_Slack__slack_search_users
---

# Ad-hoc Deal Flow Screen

## Team member → thesis mapping

| Thesis | Member | User ID |
|---|---|---|
| Future of Autonomous Work | Daria Gneusheva | `U0AA0044W1K` |
| Future of Autonomous Work | Omar Hedeya | `U0A9MUM30AK` |
| Fintech | Malin Posern | `U0A9MUWM77Z` |
| Fintech | Marjorie Lengereau | `U0AAGADJCQ1` |
| Global Supply Chain | Philipp Werner | `U0A9MUWK50X` |
| Global Supply Chain | Oskar Lingk | `U0AA1BDG7D4` |
| European Resilience | Jack Wang | `U0A9X1FNV19` |
| European Resilience | Miha Pavlovic | `U0A9X1DAVTM` |
| Surf and Turf | Ciara Gumsheimer | `U0AA004TEQ5` |

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

All core team member IDs are hardcoded in the table above — use them directly. Only call `slack_search_users` for non-team members (e.g. action item owners outside the core team). Format as `<@USERID>`.

---

## Step 7 — Compose unified list

Companies and profiles go into one single message, merged by thesis. Within each thesis section, list companies first, then profiles — but both use the same bullet format.

```
**Ad-hoc Deal Flow Screen — [Month D, YYYY]**

**Future of Autonomous Work** <@DARIA_ID> <@OMAR_ID>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**Fintech** <@MALIN_ID> <@MARJORIE_ID>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**Global Supply Chain** <@PHILIPP_ID> <@OSKAR_ID>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**European Resilience** <@JACK_ID> <@MIHA_ID>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

**Surf and Turf** <@CIARA_ID>
- <https://company.com|CompanyName> — One-sentence description.
- <https://www.linkedin.com/in/slug/|Full Name> — background context note.

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
- Company entries with no URL: plain name only, no link; use description if provided in the original list
- Profile entries with no LinkedIn URL: plain `Full Name` only
- Action item format: `| _Action: <@USERID>_` appended at end of line
- Only include sections that have entries; only include Flagged if there are flags
- If the message exceeds ~4000 characters, split into Part 1 / Part 2 with _(continued)_ on the second

---

## Step 8 — Post to #automation-tests

Post the unified message to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then output to terminal:
> "Screened [N] companies + [M] profiles → posted to #automation-tests."

---

## Step 9 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 10 — Learn from corrections (only if user provides them)

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
