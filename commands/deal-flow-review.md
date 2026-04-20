---
description: Pull deal flow from #deal-flow for a specific person and date range, and surface an organized review
allowed-tools: mcp__claude_ai_Slack__slack_read_channel, mcp__claude_ai_Slack__slack_read_thread, mcp__claude_ai_Slack__slack_search_users, mcp__claude_ai_Slack__slack_send_message
---

# Deal Flow Review

## Team member → thesis mapping
| Thesis | Member | User ID |
|---|---|---|
| Future of Autonomous Work | Daria Gneusheva | `U0AA0044W1K` |
| Future of Autonomous Work | Omar Hedeya | `U0A9MUM30AK` |
| Fintech | Malin Posern | `U0A9MUWM77Z` |
| Fintech | Marjorie Lengereau | `U0AAGADJCQ1` |
| European Resilience | Jack Wang | `U0A9X1FNV19` |
| European Resilience | Miha Pavlovic | `U0A9X1DAVTM` |
| Global Supply Chain | Philipp Werner | `U0A9MUWK50X` |
| Global Supply Chain | Oskar Lingk | `U0AA1BDG7D4` |
| Surf and Turf | Ciara Gumsheimer | `U0AA004TEQ5` |

---

## Thesis routing table
| What they build | Thesis |
|---|---|
| AI agents, orchestration, LLM infrastructure, dev tools, enterprise AI-native SaaS, robotics (software/AI-architecture-first only — see hardcoded rules below) | Future of Autonomous Work |
| Fintech, payments, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | Fintech |
| Defense, hardware, chips, non-GNSS navigation, industrial security, cloud infrastructure | European Resilience |
| Supply chain, logistics, manufacturing, materials | Global Supply Chain |
| Health, biotech, edtech, consumer, gaming, fitness, energy, creator, construction, agriculture | Surf and Turf |

**Critical routing test:** Is this company *building* AI, or *using* AI for a specific domain?
- Building AI tools / infrastructure / agents → Future of Autonomous Work
- Using AI to solve a domain problem → route to that domain's thesis

**Hardcoded routing rules:**
- Cybersecurity (pentesting, infosec, security tooling) → Future of Autonomous Work, not European Resilience
- AI sales tools (commissions, sales enablement, revenue ops) → Surf and Turf, not Fintech
- Blockchain / crypto / web3 → Fintech, not Future of Autonomous Work
- Energy companies → Surf and Turf by default. If software-based → action item for Oskar Lingk. If hardware-based → action item for Miha Pavlovic.
- Robotics (software/AI-first) — foundation models for robotics, physical intelligence, robot OS, AI frameworks → Future of Autonomous Work
- Robotics (hardware/industrial/applied) — robot hardware, manufacturing automation, applied robotics verticals → Global Supply Chain | action item for Miha Pavlovic (defense robotics remains European Resilience — the existing defense rule takes precedence)

---

## Step 1 — Gather inputs

If not already provided in the arguments, ask the following two questions **separately and in order**:

**Question 1 — Name:**
Ask: "Who should I pull deal flow for?" — accept any free-text response (first name, full name, or nickname). Do not suggest or list any team member names. Once provided, use `slack_search_users` to resolve the full name and Slack user ID. Note their first name (for output headers) and look up which thesis they belong to using the team member → thesis mapping table above.

**Question 2 — Date range:**
After confirming the name, present exactly these three options:

```
Which date range?
  1. This week (Monday to today)
  2. Last 2 weeks (last Monday to today)
  3. Custom — enter your own dates
```

If they pick 1 or 2, calculate the range automatically based on today's date (Berlin/CET timezone). If they pick 3, ask them to type the start and end dates freely.

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
