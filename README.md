# Project A — Claude Commands

Custom Claude Code slash commands for the Project A investment team.

---

## Commands

| Command | What it does |
|---|---|
| `/daily-dealflow` | Pulls new #deal-flow Slack messages, routes by thesis, and posts a Daily Dealflow Summary to #automation-tests |
| `/net-new-affinity` | Takes Affinity scraper results and generates the Net New to Affinity post to #automation-tests |
| `/deal-flow-review` | Pulls deal flow for a specific team member + date range and sends a formatted DM to their Slack |
| `/investment-team-dealflow-meetings` | Formats a Granola meeting transcript into ready-to-send Gmail email summaries |
| `/evertrace-signals` | Reads an Evertrace CSV export, routes companies by thesis, and posts a signal digest to Slack |

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

That's it. Open a new Claude Code session and `/daily-dealflow`, `/net-new-affinity`, `/deal-flow-review`, `/investment-team-dealflow-meetings`, and `/evertrace-signals` will be available.

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

### `/daily-dealflow`
Pulls every new message from #deal-flow since the last Daily Dealflow post (auto-anchored), routes all deals to the correct thesis owner, captures any Slack-explicit action items, enriches entries missing a description, and posts a Daily Dealflow Summary + raw Affinity check list to #automation-tests. Can be run manually or scheduled to run automatically at 7pm Berlin time.

**Flow:** Auto-anchor → Pull Slack → Parse + route entries → Capture action items → Enrich missing descriptions → Post Summary + Affinity list to #automation-tests

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

## Repo structure

```
project-a-claude-commands/
├── README.md
└── commands/
    ├── daily-dealflow.md
    ├── net-new-affinity.md
    ├── deal-flow-review.md
    ├── evertrace-signals.md
    └── investment-team-dealflow-meetings.md
```

---

## Questions / issues

Ping Kieran on Slack.
