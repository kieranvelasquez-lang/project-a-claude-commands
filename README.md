# Project A — Claude Commands

Custom Claude Code slash commands for the Project A investment team.

---

## Commands

| Command | What it does |
|---|---|
| `/morning-recap` | Pulls new #deal-flow Slack messages, routes by thesis, and posts the Morning Recap to #automation-tests |
| `/net-new-affinity` | Takes Affinity scraper results and generates the Net New to Affinity post to #automation-tests |
| `/deal-flow-review` | Pulls deal flow for a specific team member + date range and sends a formatted DM to their Slack |
| `/investment-team-dealflow-meetings` | Formats a Granola meeting transcript into ready-to-send Gmail email summaries |
| `/evertrace-signals` | Reads an Evertrace CSV export, routes companies by thesis, and posts a signal digest to Slack |
| `/dealflow-retro-newsletter` | Bi-weekly European funding round retro — imports Crunchbase Pro CSV, cross-references Affinity, captures pass reasons, outputs a formatted HTML email |

---

## Prerequisites

Every team member needs:

1. **Claude Code** — install at [claude.ai/download](https://claude.ai/download) and log in with your Anthropic account
2. **Slack MCP** — the Slack integration that lets Claude read and post to Slack. Set this up via Claude Code's MCP settings (ask Kieran for the config)

---

## Installation

**5-minute setup — run these commands in Terminal:**

```bash
# 1. Clone the repo
git clone https://github.com/project-a/project-a-claude-commands.git ~/project-a-claude-commands

# 2. Symlink the commands into your Claude config
#    (backs up any existing commands folder first)
[ -d ~/.claude/commands ] && mv ~/.claude/commands ~/.claude/commands.bak
ln -s ~/project-a-claude-commands/commands ~/.claude/commands

# 3. Create the memory directory (where corrections are stored)
mkdir -p ~/.claude/memory
```

That's it. Open a new Claude Code session and `/morning-recap`, `/net-new-affinity`, `/deal-flow-review`, `/investment-team-dealflow-meetings`, `/evertrace-signals`, and `/dealflow-retro-newsletter` will be available.

---

## Getting updates

When commands are updated, pull the latest and your symlink picks it up automatically:

```bash
cd ~/project-a-claude-commands && git pull
```

---

## How corrections memory works

Each command learns from your feedback and stores corrections in `~/.claude/memory/`:

- `daily-dealflow-corrections.md` — thesis routing overrides (e.g. "CompanyX should be Fintech")
- `investment-team-dealflow-meetings-corrections.md` — name, company, and HQ country fixes from Granola transcripts
- `evertrace-signals-corrections.md` — theme routing overrides, name and company name corrections for Evertrace signals

These files live on **your machine only** — not in the repo. They're personal to each user and build up over time.

---

## Commands in detail

### `/morning-recap`
Pulls every new message from #deal-flow since the last Morning Recap (auto-anchored), routes all deals to the correct thesis owner, captures any Slack-explicit action items, enriches entries missing a description, and posts a Morning Recap + raw Affinity check list to #automation-tests. Runs automatically every weekday at 8am Berlin time.

**Flow:** Auto-anchor → Pull Slack → Parse + route entries → Capture action items → Enrich missing descriptions → Post Morning Recap + Affinity list to #automation-tests

**Requires:** Slack MCP

---

### `/net-new-affinity`
Run this after your Affinity scraper. Paste your found/not-found results, confirm Net New entries, do the LinkedIn double-check, and get an enriched Net New to Affinity post (with descriptions and funding info) posted to #automation-tests.

**Flow:** Paste scraper results → Confirm Net New → LinkedIn double-check → Enrich descriptions + funding → Post to #automation-tests

**Requires:** Slack MCP

---

### `/deal-flow-review`
Given a team member's name and a date range, pulls all #deal-flow messages where they were tagged (action items) or where entries match their thesis (thesis matches). Formats a clean summary and sends it as a Slack DM to that person.

**Usage:** `/deal-flow-review Malin Posern` then follow prompts for date range

**Requires:** Slack MCP

---

### `/investment-team-dealflow-meetings`
Paste a Granola meeting transcript and get back two ready-to-send email summaries — Version 1 (internal, full detail) and Version 2 (clean, compliance-facing). Output opens directly in your browser for copy-paste into Gmail.

**Usage:** `/investment-team-dealflow-meetings` then paste transcript when prompted

**Requires:** Nothing beyond Claude Code

---

### `/evertrace-signals`
Export a CSV from Evertrace, run this command, answer three questions (CSV path, week label, include stealth?), and get a clean signal digest posted to Slack. Companies are auto-routed to thesis owners. Previews in #automation-tests before posting to #deal-flow.

**Usage:** `/evertrace-signals` then follow prompts (drag CSV into terminal for path)

**Requires:** Slack MCP

---

### `/dealflow-retro-newsletter`
Bi-weekly digest of all European tech funding rounds. Import a Crunchbase Pro CSV export (filtered to Europe + date range), Claude supplements with EU-Startups WebFetch, you paste Affinity cross-reference results and any pass reasons, and Claude generates a formatted HTML email table — ready to copy into Gmail and send to the investment team and all partners.

**Usage:** `/dealflow-retro-newsletter` then follow prompts (confirm date range → export Crunchbase CSV → Affinity check → pass reasons → email opens in browser)

**Requires:** Crunchbase Pro subscription (for CSV export). Dealroom CSV can be provided alongside for better coverage if access is available.

---

## Repo structure

```
project-a-claude-commands/
├── README.md
├── commands/
│   ├── morning-recap.md
│   ├── net-new-affinity.md
│   ├── deal-flow-review.md
│   ├── evertrace-signals.md
│   ├── investment-team-dealflow-meetings.md
│   └── dealflow-retro-newsletter.md
└── memory/
    ├── morning-recap/                       ← corrections + feedback for /morning-recap
    ├── net-new-affinity/                    ← (created when first correction is learned)
    ├── deal-flow-review/                    ← (created when first correction is learned)
    ├── evertrace-signals/                   ← (created when first correction is learned)
    ├── investment-team-dealflow-meetings/   ← (created when first correction is learned)
    └── dealflow-retro-newsletter/           ← (created when first correction is learned)
```

Each skill gets its own subfolder under `memory/` as it accumulates corrections and feedback over time. Only `morning-recap/` is populated today — the others are created automatically the first time a correction is learned for that skill.

---

## Questions / issues

Ping Kieran on Slack.
