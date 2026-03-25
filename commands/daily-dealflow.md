---
description: Pull today's #deal-flow Slack messages, enrich and route them, then publish clean posts back to Slack
allowed-tools: Read, Write, WebSearch, WebFetch, mcp__claude_ai_Slack__slack_read_channel, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_send_message, mcp__claude_ai_Slack__slack_search_users
---

# Daily Dealflow Automation

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

Read `~/.claude/projects/-Users-kvelasquez-Projects/memory/daily-dealflow-corrections.md` using the Read tool.

This file is a **staging area** for new corrections that haven't yet been baked into the skill. The default routing rules below already incorporate all previously confirmed corrections.

- If the file is empty or has no entries: proceed — no delta to apply.
- If the file has entries: load them into context as overrides. Note them briefly (e.g. "1 staged correction: CompanyX → Surf and Turf"). These take precedence over default routing for any matching company name.

Do not re-read or re-explain the full routing table — just flag what's new.

---

## Step 2 — Pull from Slack

Ask the user: "What time was the last Daily Dealflow post sent? (e.g. '7:22 PM on March 16')" — or, if they already provided it, use that.

Convert the time to a Unix timestamp (remember: Project A is Berlin, CET = UTC+1, CEST = UTC+2 from late March). Use that timestamp as the `oldest` parameter when calling `slack_read_channel`.

**Never fetch the full channel history.** Always use `oldest` to fetch only messages posted after the last Daily Dealflow post sent by Kieran Velasquez.

Use `slack_read_channel` on #deal-flow (channel ID: `C0AB6LUVCN4`) with `oldest` set to the anchor timestamp.

If the user hasn't provided a timestamp and none is available from context, ask:
> "Please give me the time (and date if not today) of the last Daily Dealflow post so I can anchor from there."

For each message retrieved:
- Capture timestamp, full content, any names already assigned inline
- If a message has thread replies, use `slack_read_thread` to capture additional context (additional links, descriptions, investor info, assignee notes)

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

**First: apply staged corrections.** If the company name appears in `daily-dealflow-corrections.md`, use that thesis — do not apply default routing rules.

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

---

## Step 4 — Present raw list for Affinity check

Output a minimal list — **company name + URL only** for companies, **LinkedIn profile name + URL only** for people. No descriptions, no funding, no thesis grouping. Just what the scraper needs.

Then ask:
> "Here's today's raw list. Run your Affinity scraper against these and paste back your confirmed found/not-found results."

Wait for the user's response before proceeding.

---

## Step 5 — Affinity cross-reference (user-led)

Wait for the user to paste their confirmed found/not-found list from their Affinity scraper run.

**Critical:** The scraper has a high false-negative rate. Only entries the user's manual check explicitly confirms as "not in Affinity" go into the Net New post. Do not rely on scraper output alone — if the user hasn't manually confirmed something as absent, do not include it in Net New.

**LinkedIn profiles:** LinkedIn blocks all automated fetching (HTTP 999). Once the user confirms which profiles are Net New, output a clickable list of those LinkedIn URLs so the user can open them directly from the terminal without digging back through Slack:

> "Here are the Net New LinkedIn profiles — click through and paste a one-liner per person:
> • https://www.linkedin.com/in/slug-one/
> • https://www.linkedin.com/in/slug-two/
>
> e.g. `pierre-dorge → Co-founder Fileforge, building in stealth`"

Use whatever the user provides as the profile description — do not attempt WebFetch on LinkedIn URLs.

---

## Step 5b — Enrich Net New entries only

For each entry the user confirmed as NOT in Affinity:

### Verify description via website
Use WebFetch on the company's actual website homepage. Do **not** use the Slack message description as final — always verify or replace with what the company actually says about itself. If the site is down or returns an error, use `_No confirmed build info available._` as the description and flag it.

### Research funding
Use WebSearch to find funding info (search: company name + "funding" or "raises" + site:crunchbase.com or site:techcrunch.com or press releases).

Include whatever is findable — amount, round type, lead investor, date. Format flexibly based on what's available:
- `_Raised: $1.5M, Pre-Seed_` — amount + round
- `_Raised: $1.52M, Seed, EWOR_` — amount + round + lead investor
- `_Raised: Pre-Seed_` — round only (amount unknown)
- `_Raised: £4–5M Seed, raising Jun 2026_` — include upcoming raise info if confirmed

If no funding info is findable, **omit the Raised field entirely** — do not write `_Raised: Unknown_`.

Do not include approximate amounts without a `~` prefix (e.g. `_Raised: ~€1M, Pre-Seed_`).
Do not include months/years unless confirmed.
LinkedIn profiles do not get a Raised field.

### Flag when needed
Flag an entry only when genuinely needed:
- Broken or dead links
- Contradictory funding info (e.g. two sources disagree)
- Site down (couldn't verify description) — try WebSearch for the company before giving up
- Truly unresolvable routing (e.g. company is in stealth with zero public info AND no Slack context to infer from)

**Do not flag for ambiguous routing** — make a best-guess thesis assignment based on the founder's background, Slack context, or who it was tagged to. Only move to Flagged if you genuinely cannot assign a thesis after using all available context.

**Do not flag for missing descriptions** — try WebSearch (company name + "what does it do" or "startup") before giving up. Only flag if both WebFetch and WebSearch return nothing useful.

**Never skip entries.** Flag missing info rather than omitting.

Present the enriched Net New list, grouped by thesis, for a final review pass before composing the post.

---

## Step 6 — Resolve @mention user IDs

Before composing posts, use `slack_search_users` to look up the Slack user ID for each team member whose thesis section appears in this run's output. Format pings as `<@USERID>` in the post body — this ensures the message actually pings them rather than rendering as plain text.

---

## Step 7 — Compose Net New post

Match this format exactly — based on real posts from #deal-flow:

```
**Daily Dealflow — [Month D, YYYY] | Net New to Affinity**
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
- Only include the "Flagged for Review" block if there are actual flags
- One blank line between each thesis section
- No lead investor unless confirmed — don't guess
- Omit `| _Raised:_` entirely if no funding info is available — never write `_Raised: Unknown_`
- Do NOT include `_Sent using Claude_` in the message — the Slack MCP appends it automatically; including it causes a duplicate

### Formatting rules (CRITICAL — never deviate)

- Bold: `**double asterisk**` — the Slack MCP uses standard markdown where `**` = bold and `*` = italic. Never use single asterisk for bold or it will render as italic.
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never markdown `[text](url)` format
- @mentions: `<@USERID>` — never plain `@name` (won't ping)
- Ampersands: raw `&` — never `&amp;`
- Descriptions end with `.` before ` | _Raised:_`
- No internal commentary in entries (who's keen, who's following up, investor opinions)
- No "sourced by" attribution
- No sentence fragments — if description can't be verified, use `_No confirmed build info available._`
- Do NOT append `_Sent using Claude_` — the MCP adds this automatically; including it creates a duplicate line
- If post exceeds ~4000 characters, split into Part 1 / Part 2

---

## Step 8 — Post to #automation-tests

Post to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then output:
> "Posts are live in #automation-tests for review. Type 'post' to publish to #deal-flow, or give me corrections and I'll revise."

---

## Step 9 — Await approval

- If the user pastes corrections: revise and repost to #automation-tests. Repeat until approved.
- If the user types 'post': proceed to Step 10.

**Never post to #deal-flow before the user has approved the #automation-tests version.**

---

## Step 10 — Post to #deal-flow

Post to channel ID `C0AB6LUVCN4` using `slack_send_message`.

> **Note:** The Affinity emoji reaction step (marking processed messages in Slack) remains manual — the Slack MCP does not expose a reaction tool.

---

## Step 11 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 12 — Learn from corrections (only if user provides them)

For each routing correction provided:

1. Determine if this is **company-specific** (e.g. "AcmeCorp → Fintech") or **general** (e.g. "AI sales tools → Surf and Turf").
   - **General rules** should be baked directly into the hardcoded routing rules in Step 3b of this skill — ask Kieran to confirm before editing.
   - **Company-specific corrections** go into the staging file.

2. For company-specific corrections, check if the company already exists in `~/.claude/projects/-Users-kvelasquez-Projects/memory/daily-dealflow-corrections.md`. If the file doesn't exist, create it with this header:

```markdown
# Daily Dealflow — Corrections Memory

## Staged Corrections (not yet baked into skill)
<!-- Format: CompanyName → Thesis | added YYYY-MM-DD -->
```

3. Append only new entries (skip duplicates):
```
- CompanyName → Thesis Name | added YYYY-MM-DD
```

4. Tell the user:
> "Learned [N] new correction(s): [list]. Staged for future runs. Let me know if any should be baked into the skill as a permanent rule."
