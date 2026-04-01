---
description: Pull yesterday's #deal-flow Slack messages, enrich and route them, then publish the Morning Recap to #automation-tests
allowed-tools: Read, Write, WebSearch, WebFetch, mcp__claude_ai_Slack__slack_read_channel, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_send_message, mcp__claude_ai_Slack__slack_search_users
---

# Morning Recap

## Team member Slack display names
Used in section headers. These are exact Slack display names — look up user IDs via `slack_search_users` before composing posts so @mentions actually ping people.

| Thesis | Members |
|---|---|
| Future of Autonomous Work | Daria Gneusheva, Omar Hedeya |
| Fintech | Malin Posern, Marjorie Lengereau |
| European Resilience | Jack Wang, Miha Pavlovic |
| Global Supply Chain | Philipp Werner, Oskar Lingk |
| Surf and Turf | Ciara Gumsheimer |

---

## Step 1 — Check corrections delta

Read `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md` using the Read tool.

This file is a **staging area** for new corrections that haven't yet been baked into the skill. The default routing rules below already incorporate all previously confirmed corrections.

- If the file is empty or has no entries: proceed — no delta to apply.
- If the file has entries: load them into context as overrides. Note them briefly (e.g. "1 staged correction: CompanyX → Surf and Turf"). These take precedence over default routing for any matching company name.

Do not re-read or re-explain the full routing table — just flag what's new.

---

## Step 2 — Pull from Slack (auto-anchor)

**Do not ask the user for a timestamp.** Anchor automatically:

1. Call `slack_read_channel` on #deal-flow (channel ID: `C0AB6LUVCN4`) with `oldest` set to 48 hours ago (current Unix timestamp minus 172800).
2. Scan the returned messages for the most recent one by Kieran Velasquez that begins with `**Morning Recap —`. Use that message's `ts` value as the anchor.
3. If no such message is found in the 48-hour window, fall back and ask: "I couldn't find your last Morning Recap post. What time was it sent? (e.g. '7:22 PM on March 16')"

Once the anchor `ts` is established, call `slack_read_channel` again with `oldest` set to that `ts` to fetch only messages posted after the last Morning Recap.

**Exclusion rule:** The anchor message is a boundary marker only — do not parse its text for company or LinkedIn entries. Do not read or process its thread replies (these will contain the Net New to Affinity post going forward and should be ignored). Only process messages with `ts` strictly greater than the anchor `ts`.

Remember: Project A is Berlin — CET = UTC+1, CEST = UTC+2 from late March.

For each message retrieved:
- Capture timestamp, full content, sender display name, any names already assigned inline
- If a message has thread replies, use `slack_read_thread` to capture additional context (links, descriptions, investor info, assignee notes, action items)

---

## Step 3 — Parse each entry (no enrichment yet)

For every company or LinkedIn profile mentioned, do the following:

### 3a. Identify type
Is this a company deal or a LinkedIn profile link?

### 3b. Extract inline data only
Pull directly from Slack — no WebFetch or WebSearch yet:
- Company/person name
- URL from Slack message
- Inline description from Slack message (verbatim or lightly cleaned)
- Funding info only if explicitly mentioned in Slack
- Thesis routing — apply staged corrections first, then default routing table below

**First: apply staged corrections.** If the company name appears in `morning-recap-corrections.md`, use that thesis — do not apply default routing rules.

**Otherwise, use default routing:**

| What they build | Thesis | Team |
|---|---|---|
| AI agents, orchestration, LLM infrastructure, dev tools, enterprise AI-native SaaS | Future of Autonomous Work | Daria Gneusheva, Omar Hedeya |
| Fintech, payments, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | Fintech | Malin Posern, Marjorie Lengereau |
| Defense, hardware, chips, non-GNSS navigation, industrial security, cloud infrastructure | European Resilience | Jack Wang, Miha Pavlovic |
| Supply chain, logistics, manufacturing, materials, robotics | Global Supply Chain | Philipp Werner, Oskar Lingk |
| Health, biotech, edtech, consumer, gaming, fitness, energy, creator, construction, agriculture | Surf and Turf | Ciara Gumsheimer |

**Critical routing test:** Is this company *building* AI, or *using* AI for a specific domain?
- Building AI tools / infrastructure / agents → Future of Autonomous Work
- Using AI to solve a domain problem (e.g. AI for insurance, AI for logistics, AI for plant breeding) → route to that domain's thesis

Never default to Future of Autonomous Work without applying this test first.

**Hardcoded routing rules (confirmed corrections, already baked in):**
- Cybersecurity (pentesting, infosec, security tooling) → Future of Autonomous Work, not European Resilience
- AI sales tools (commissions, sales enablement, revenue ops) → Surf and Turf, not Fintech
- Blockchain / crypto / web3 startups → Fintech, not Future of Autonomous Work (even if building observability or tooling for blockchain networks)

### 3c. Flag unknowns
Flag entries with no URL and no name, but still capture them — never skip entries.

### 3d. Capture inline assignments and action items

After routing, check for explicit Slack-level assignments that override default routing:

**In the original message:**
- Is a specific team member @mentioned alongside the deal? If yes, route to that person's thesis.

**In thread replies:**
- Look for language like "tagging this to X", "passing to X", "X will reach out", "X is following up", "pinging X on this".
- Track the full chain: if A assigns to B, and B reassigns to C, follow to C.

**Routing override rules:**
- If the final explicit assignee is on a different thesis than default routing → override the thesis to match the assignee.
- If tagger and thread responder are on the **same thesis team**, no routing change needed (e.g. Marjorie sends + tags Miha, Jack also responds → still European Resilience).
- If a cross-thesis reassignment exists (e.g. Miha tags Daria, Daria passes to Marjorie) → route to final assignee's thesis (Fintech), and flag the action item owner.

**Action item flag:**
Record who owns the action item (if anyone). This will be displayed inline in the Morning Recap as `| _Action: <@USERID>_`.

If no explicit action item exists, leave this field blank — do not invent one.

---

## Step 4 — Enrich entries with no Slack description

For any entry where the Slack message included **no description**, enrich before composing the Morning Recap:

**Companies:**
- Use WebFetch on the company's actual website homepage to pull a one-sentence description.
- If WebFetch fails, try WebSearch (company name + "startup" or "what does it do").
- If both fail, flag the entry — do not invent a description.
- Do **not** research funding here — that belongs in `/net-new-affinity`.

**LinkedIn profiles:**
- LinkedIn blocks automated fetching. If no description was in Slack, omit the description in the Summary — do not attempt WebFetch.

---

## Step 5 — Resolve @mention user IDs

Before composing the post, use `slack_search_users` to look up the Slack user ID for each team member whose thesis section appears in this run's output. Format pings as `<@USERID>` in the post body — this ensures the message actually pings them rather than rendering as plain text.

---

## Step 6 — Compose Morning Recap

Match this format exactly:

```
**Morning Recap — [Month D, YYYY]**
_Review of yesterday's dealflow · [Month D, YYYY]_

**Future of Autonomous Work** <@DARIA_ID> <@OMAR_ID>
- <https://company.com|CompanyName> — One-sentence description.
- <https://company.com|CompanyName2> — One-sentence description. | _Action: <@USERID>_
- <https://linkedin.com/in/handle|Full Name> — One-sentence bio/context.

**Global Supply Chain** <@PHILIPP_ID> <@OSKAR_ID>
- <https://company.com|CompanyName> — One-sentence description.

**Fintech** <@MALIN_ID> <@MARJORIE_ID>
- <https://company.com|CompanyName> — One-sentence description. | _Action: <@MARJORIE_ID>_

**Surf and Turf** <@CIARA_ID>
- <https://company.com|CompanyName> — One-sentence description.

**European Resilience** <@JACK_ID> <@MIHA_ID>
- <https://company.com|CompanyName> — One-sentence description.

---

**⚠️ Flagged for Review**
- CompanyName — reason
```

Rules:
- Include **all** entries from the day — not just Net New. Companies already in Affinity belong here too.
- Only include thesis sections that have entries.
- Only include "Flagged for Review" if there are actual flags.
- One blank line between each thesis section.
- Do **not** include `| _Raised:_` in the Recap — funding info belongs in `/net-new-affinity` only.
- Action item format: `| _Action: <@USERID>_` appended at end of line, only when a confirmed action item owner exists.
- LinkedIn profiles with no Slack description: show link and name only (`- <https://linkedin.com/in/handle|Full Name>`), no description fragment.
- Do NOT include `_Sent using Claude_` — the MCP appends it automatically.
- If post exceeds ~4000 characters, split into Part 1 / Part 2.

### Formatting rules (CRITICAL — never deviate)

- Bold: `**double asterisk**`
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never markdown `[text](url)` format
- @mentions: `<@USERID>` — never plain `@name`
- Ampersands: raw `&` — never `&amp;`
- No internal commentary in entries (who's keen, who's following up, investor opinions)
- No "sourced by" attribution
- Do NOT append `_Sent using Claude_`

---

## Step 7 — Post to #automation-tests

Post the Morning Recap to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then, as a **second message** to the same channel, post the Affinity Check List. Include **every** entry from the Morning Recap (all thesis sections, including LinkedIn profiles) — not a partial subset. Format each entry on its own line:

```
• CompanyName — https://url.com
• FullName (LinkedIn) — https://linkedin.com/in/handle
```

Use plain URLs — **not** Slack `<url|text>` link syntax. Use `•` bullet prefix for every entry. Do not group by thesis. Do not include descriptions. Follow the list with one blank line, then:

> "Morning Recap is live above. Run your Affinity scraper against this list, then run `/net-new-affinity` and paste your results to generate the Net New post."

Then output to the terminal:
> "Morning Recap and Affinity list are live in #automation-tests. Copy the Recap to #deal-flow when ready, then run `/net-new-affinity` with your scraper results."

---

## Step 8 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 9 — Learn from corrections (only if user provides them)

For each routing correction provided:

1. Determine if this is **company-specific** (e.g. "AcmeCorp → Fintech") or **general** (e.g. "AI sales tools → Surf and Turf").
   - **General rules** should be baked directly into the hardcoded routing rules in Step 3b of this skill — ask Kieran to confirm before editing.
   - **Company-specific corrections** go into the staging file.

2. For company-specific corrections, check if the company already exists in `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md`. If the file doesn't exist, create it with this header:

```markdown
# Morning Recap — Corrections Memory

## Staged Corrections (not yet baked into skill)
<!-- Format: CompanyName → Thesis | added YYYY-MM-DD -->
```

3. Append only new entries (skip duplicates):
```
- CompanyName → Thesis Name | added YYYY-MM-DD
```

4. Tell the user:
> "Learned [N] new correction(s): [list]. Staged for future runs. Let me know if any should be baked into the skill as a permanent rule."
