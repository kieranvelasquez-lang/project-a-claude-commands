---
description: Pull deal flow from #deal-flow for a specific person and date range, and surface an organized review
allowed-tools: mcp__claude_ai_Slack__slack_read_channel, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_search_users, mcp__claude_ai_Slack__slack_send_message
---

# Deal Flow Review

## Team member → thesis mapping
| Thesis | Members |
|---|---|
| Future of Autonomous Work | Daria Gneusheva, Omar Hedeya |
| Fintech | Malin Posern, Marjorie Lengereau |
| European Resilience | Jack Wang, Miha Pavlovic |
| Global Supply Chain | Philipp Werner, Oskar Lingk |
| Surf and Turf | Ciara Gumsheimer |

---

## Thesis routing table
| What they build | Thesis |
|---|---|
| AI agents, orchestration, LLM infrastructure, dev tools, enterprise AI-native SaaS | Future of Autonomous Work |
| Fintech, payments, insurance, compliance, legal, payroll, tax | Fintech |
| Defense, hardware, chips, non-GNSS navigation, industrial security, cloud infrastructure | European Resilience |
| Supply chain, logistics, manufacturing, materials, robotics, construction, agriculture | Global Supply Chain |
| Health, biotech, edtech, consumer, gaming, fitness, energy, creator | Surf and Turf |

**Critical routing test:** Is this company *building* AI, or *using* AI for a specific domain?
- Building AI tools / infrastructure / agents → Future of Autonomous Work
- Using AI to solve a domain problem → route to that domain's thesis

**Energy sub-rule:** Energy companies → Surf and Turf. Note inline action tag when classifying:
- Software-only energy (SaaS, platform, software, data, app, digital) → Surf and Turf + Action: Oskar Lingk
- Hardware-only energy (physical infrastructure, hardware, devices, equipment) → Surf and Turf + Action: Miha Pavlovic
- Mixed or unclear → Surf and Turf, no action tag

---

## Step 1 — Gather inputs

If not already provided in the arguments, ask:
1. **Who to pull for** — full name (e.g. "First Last")
2. **Date range** — start date and end date (e.g. "March 2 to March 13, 2026")

Once you have the name, use `slack_search_users` to look up the person's Slack user ID. You'll need this to detect `<@USERID>` mentions in messages.

Also note their first name (for output headers) and look up which thesis they belong to using the team member → thesis mapping table above.

---

## Step 2 — Fetch messages

Convert the date range to Unix timestamps. The team operates in Berlin/CET timezone (UTC+1 standard, UTC+2 during CEST from late March). Convert accordingly.

Use `slack_read_channel` on #deal-flow (channel ID: `C0AB6LUVCN4`) with:
- `oldest` = Unix timestamp for start of date range (beginning of day, Berlin time)
- `latest` = Unix timestamp for end of date range (end of day, Berlin time)

For every message that has thread replies, use `slack_read_thread` to capture the full thread context — additional tags, notes, investor comments, and assignee callouts often appear in replies rather than the top-level message.

---

## Step 3 — Classify each entry

For each message and thread, determine which of the following applies to the target person:

**Action Item** — the message:
- Contains `<@USERID>` of the target person, OR
- Contains a plaintext `@FirstName` or `@FullName` reference, OR
- Explicitly mentions the person in a tagging or routing context (e.g. "for Marjorie", "tagging Malin")

**Thesis Match** — the entry:
- Is relevant to the person's thesis area (use the routing table above)
- Was NOT directly tagged to the person

**Flagged** — anything anomalous:
- Conflicting information (e.g. same company tagged to two theses)
- Unclear or missing company name
- Entry references the person in an ambiguous way

Do not double-count: if an entry qualifies as both an Action Item and a Thesis Match, classify it as an Action Item only.

---

## Step 4 — Format and output

First print the output to the terminal as a preview. Then send the Slack-formatted version as a DM to the reviewed person (see Step 5).

Use this exact format:

```
Deal Flow for [Full Name] — [Start Date] to [End Date]
Pulled from #deal-flow · N entries total: X action items, Y thesis matches

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Action Items (directly tagged to [First Name])
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• [Company/Person Name] — [One-sentence description]. | Raised: [amount] | Sourced by: [name] · [Day, Mon DD]
  ↳ Action: [why they were tagged — verbatim or summarized from Slack]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Thesis Matches ([Thesis Name] — not directly tagged)
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• [Company Name] — [One-sentence description]. | Sourced by: [name] · [Day, Mon DD]

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Flagged
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

• [Company] — [reason]

---
Your to-do list:
1. [Company] — [brief action needed]
2. [Company] — [brief action needed]
```

### Formatting rules

- **Action Items** always come first, sorted newest-first within each date group
- **Thesis Matches** second
- **Flagged** third (omit section entirely if nothing to flag)
- **To-do list** summarizes only Action Items — group batch tagging from a single list where sensible
- Descriptions are one sentence max — pull from Slack context; do not enrich via web
- Include `Sourced by:` and date on every entry
- If funding amount was explicitly mentioned in Slack, include it; if not, omit the Raised field entirely (do not write "Raised: Unknown")
- No internal opinions, no investor reactions, no thesis routing markers in the output
- Use descriptions verbatim or lightly cleaned from Slack — no fabrication

---

## Step 5 — Send to Slack DM

Using the person's Slack user ID (retrieved in Step 1), send the review as a DM to themselves using `slack_send_message` with `channel_id` set to their user ID. Sending to a user's own ID delivers the message to their "DM with yourself" (Slack saved messages).

Format the Slack message using Slack mrkdwn — *not* terminal unicode. Key differences from the terminal version:

- **Section headers**: use `**double asterisk bold**` instead of ━━━ dividers
  e.g. `**Action Items (directly tagged to [First Name])**`
- **Company/profile links**: always use Slack link syntax `<URL|display text>` — never markdown `[text](url)`
  e.g. `<https://company.com|Company Name>` or `<https://linkedin.com/in/xyz|First Last>`
- **Bullets, ↳, and pipe separators** (`•`, `↳`, `|`) render fine in Slack — keep them
- **Ampersands**: always raw `&` — never `&amp;`
- **Blank lines**: one blank line between each thematic section
- **Raised / Sourced by / date** — keep inline as in the terminal version
- **To-do list**: keep as a numbered list under `**Your to-do list:**`
- End the message with the summary line: `[N] action items · [M] thesis matches · Pulled from #deal-flow · [Start Date]–[End Date]`
- **Message length**: Slack has a ~4000 character practical limit. If the message exceeds this, split into two messages: Part 1 (Action Items + Flagged) and Part 2 (Thesis Matches + to-do list), with the title repeated and marked _(continued)_ on the second.

After sending, confirm to the terminal: `Sent to [First Name]'s Slack DMs.`

---

## Step 6 — Summary line

After the full output, print this final line:

```
[N] action items · [M] thesis matches · Pulled from #deal-flow · [Start Date]–[End Date]
```
