# Project A — Claude Commands

Custom Claude Code slash commands for the Project A investment team.

---

## Commands

| Command | What it does |
|---|---|
| `/daily-dealflow` | Pulls new #deal-flow Slack messages, enriches them, routes by thesis, and posts back to Slack |
| `/deal-flow-review` | Pulls deal flow for a specific team member + date range and sends a formatted DM to their Slack |
| `/weekly-dealflow` | Formats a Granola meeting transcript into ready-to-send Gmail email summaries |

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

That's it. Open a new Claude Code session and `/daily-dealflow`, `/deal-flow-review`, and `/weekly-dealflow` will be available.

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
- `weekly-dealflow-corrections.md` — name, company, and HQ country fixes from Granola transcripts

These files live on **your machine only** — not in the repo. They're personal to each user and build up over time.

---

## Commands in detail

### `/daily-dealflow`
Pulls every new message from #deal-flow since the last Daily Dealflow post, enriches companies via web search, routes them to the correct thesis owner, and posts a clean Net New to Affinity summary back to Slack.

**Flow:** Pull Slack → Parse entries → Present raw list for Affinity check → Enrich Net New only → Preview in #automation-tests → Approve → Post to #deal-flow

**Requires:** Slack MCP

---

### `/deal-flow-review`
Given a team member's name and a date range, pulls all #deal-flow messages where they were tagged (action items) or where entries match their thesis (thesis matches). Formats a clean summary and sends it as a Slack DM to that person.

**Usage:** `/deal-flow-review Malin Posern` then follow prompts for date range

**Requires:** Slack MCP

---

### `/weekly-dealflow`
Paste a Granola meeting transcript and get back two ready-to-send email summaries — Version 1 (internal, full detail) and Version 2 (clean, compliance-facing). Output opens directly in your browser for copy-paste into Gmail.

**Usage:** `/weekly-dealflow` then paste transcript when prompted

**Requires:** Nothing beyond Claude Code

---

## Repo structure

```
project-a-claude-commands/
├── README.md
└── commands/
    ├── daily-dealflow.md
    ├── deal-flow-review.md
    └── weekly-dealflow.md
```

---

## Questions / issues

Ping Kieran on Slack.
