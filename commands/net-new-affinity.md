---
description: Generate the Net New to Affinity post from Affinity scraper results and post to #automation-tests
allowed-tools: Read, mcp__claude_ai_Slack__slack_send_message
---

# Net New to Affinity

Run this after `/morning-recap` has posted the Morning Recap and Affinity list to #automation-tests, and after you have run your Affinity scraper against that list.

## Thesis routing table

| What they build | Thesis |
|---|---|
| AI agents, orchestration, LLM infrastructure, dev tools, enterprise AI-native SaaS, robotics | Future of Autonomous Work |
| Fintech, payments, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | Fintech |
| Defense, hardware, chips, non-GNSS navigation, industrial security, cloud infrastructure | European Resilience |
| Supply chain, logistics, manufacturing, materials | Global Supply Chain |
| Health, biotech, edtech, consumer, gaming, fitness, energy, creator, construction, agriculture | Surf and Turf |

**Hardcoded routing rules:**
- Cybersecurity → Future of Autonomous Work, not European Resilience
- AI sales tools → Surf and Turf, not Fintech
- Blockchain / crypto / web3 → Fintech, not Future of Autonomous Work

---

## Step 1 — Load corrections

Read `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md`. Load any staged corrections as overrides for thesis routing. Note briefly if any exist.

---

## Step 2 — Accept Affinity scraper results

Ask:
> "Paste your Affinity scraper results (found / not-found list)."

Wait for the user's response before proceeding.

---

## Step 3 — Confirm Net New entries

**Critical:** The scraper has a high false-negative rate. Only entries the user's manual check explicitly confirms as "not in Affinity" proceed to the Net New post. Do not rely on scraper output alone.

Separate the list into:
- **Confirmed Net New** — user explicitly says not in Affinity
- **Found / uncertain** — skip these; do not include in Net New post

---

## Step 4 — Compose Net New to Affinity post

Match this format exactly:

```
**Net New to Affinity — [Month D, YYYY]**
_Entries not yet tracked in Affinity · [Month D, YYYY]_

**Future of Autonomous Work**
- <https://company.com|CompanyName>
- <https://linkedin.com/in/handle|Full Name>

**Global Supply Chain**
- <https://company.com|CompanyName>

**Fintech**
- <https://company.com|CompanyName>

**Surf and Turf**
- <https://company.com|CompanyName>

**European Resilience**
- <https://company.com|CompanyName>
```

Rules:
- Only include thesis sections that have entries
- One blank line between each thesis section
- No descriptions, no @mentions, no funding info
- Links only — `<https://url|display name>` format
- Do NOT include `_Sent using Claude_`
- If post exceeds ~4000 characters, split into Part 1 / Part 2

### Formatting rules

- Bold: `**double asterisk**`
- Italic: `_underscore_`
- Links: `<https://url|display text>` — never markdown `[text](url)` format
- Ampersands: raw `&` — never `&amp;`

---

## Step 5 — Post to #automation-tests

Post to channel ID `C0AKKPK3J1K` using `slack_send_message`.

Then output:
> "Net New post is live in #automation-tests. Copy it to #deal-flow when ready."

---

## Step 6 — Ask for routing corrections

Ask:
> "Were any thesis routings wrong? If yes, tell me: 'CompanyName should be [Thesis]' and I'll learn it. Type 'no' to finish."

---

## Step 7 — Learn from corrections (only if user provides them)

For each routing correction provided:

1. Determine if this is **company-specific** or **general**.
   - **General rules** → ask Kieran to confirm before baking into the hardcoded routing rules above.
   - **Company-specific corrections** → staging file.

2. Check if company already exists in `~/.claude/projects/-Users-kvelasquez-Projects/memory/morning-recap-corrections.md`. If file doesn't exist, create it with:

```markdown
# Morning Recap — Corrections Memory

## Staged Corrections (not yet baked into skill)
<!-- Format: CompanyName → Thesis | added YYYY-MM-DD -->
```

3. Append only new entries:
```
- CompanyName → Thesis Name | added YYYY-MM-DD
```

4. Tell the user:
> "Learned [N] new correction(s): [list]. Staged for future runs. Let me know if any should be baked into the skill as a permanent rule."
