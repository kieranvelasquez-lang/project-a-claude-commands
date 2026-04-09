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
| `/dealflow-retro-newsletter` | Bi-weekly European funding round retro — imports Crunchbase Pro CSV, cross-references Affinity, outputs a formatted HTML email with direct Affinity entry links |
| `/dealflow-screener` | Paste any ad-hoc deal flow list — Claude enriches and routes companies (Part 1), then handles LinkedIn profiles with a user context step before routing (Part 2), posting both to #automation-tests |

---

## Setup guide

Follow these four steps in order. The whole process takes about 15 minutes.

---

### Step 1 — Install Claude Code

1. Go to [claude.ai/download](https://claude.ai/download) and download the Mac desktop app
2. Open it and sign in with your Anthropic account (create one at [claude.ai](https://claude.ai) if you don't have one yet — you need a paid Claude Pro or Team plan)
3. Once you're in, open a terminal window inside Claude Code (or use your system Terminal — both work)

---

### Step 2 — Set up GitHub access (Personal Access Token)

This repo is private, so you need to authenticate with GitHub before you can clone it. You do this once and it's saved.

**Create a Personal Access Token (PAT):**

1. Go to [github.com/settings/tokens](https://github.com/settings/tokens) (you must be logged into GitHub)
2. Click **"Generate new token" → "Generate new token (classic)"**
3. Give it a name (e.g. `project-a-claude-commands`)
4. Set expiration to **90 days** (or "No expiration" if you prefer)
5. Under **Scopes**, tick **`repo`** (this gives read access to private repos)
6. Click **"Generate token"** at the bottom
7. Copy the token — it starts with `ghp_...` — you won't see it again after you leave the page

**Configure git to use your token:**

Run this in Terminal, replacing `YOUR_GITHUB_USERNAME` and `YOUR_TOKEN`:

```bash
git config --global credential.helper store
echo "https://YOUR_GITHUB_USERNAME:YOUR_TOKEN@github.com" >> ~/.git-credentials
```

---

### Step 3 — Clone the repo and link the commands

Run these commands in Terminal one at a time:

```bash
# 1. Clone the repo into your home directory
git clone https://github.com/kieranvelasquez-lang/project-a-claude-commands.git ~/Projects/project-a-claude-commands
```

```bash
# 2. Create the Claude commands directory if it doesn't exist
mkdir -p ~/.claude/commands
```

```bash
# 3. Copy the commands into your Claude config
#    (or run this again any time you want to pull the latest versions)
cp ~/Projects/project-a-claude-commands/commands/*.md ~/.claude/commands/
```

```bash
# 4. Create the memory directory (where Claude stores your personal corrections)
mkdir -p ~/.claude/memory
```

Open a new Claude Code session. You should now see `/morning-recap`, `/net-new-affinity`, `/deal-flow-review`, `/investment-team-dealflow-meetings`, `/evertrace-signals`, `/dealflow-retro-newsletter`, and `/dealflow-screener` in the slash command menu.

---

### Step 4 — Connect Claude to Slack (Slack MCP)

Several commands read from and post to the Project A Slack workspace. To enable this, you need to connect **your own Slack account** to Claude — each person does this individually with their own credentials.

**What you're setting up:** An MCP (Model Context Protocol) server that gives Claude permission to read channels and send messages in Slack on your behalf. Think of it like granting Claude the same access you have in Slack.

**Steps:**

1. Open Claude Code and go to **Settings → MCP Servers**
2. Click **"Add MCP Server"**
3. Select **Slack** from the list of available integrations (or add it manually — ask Kieran for the server config if it doesn't appear)
4. Follow the OAuth flow to authorise with your Project A Slack account (`@yourname@project-a.vc`)
5. When asked which permissions to grant, approve all that are requested — Claude needs read and write access to channels and DMs

> **Note:** You are connecting your own Slack account. Claude will post and read messages as you — the same way you would if you were doing it manually. Kieran's Slack account is not involved.

**Verify it's working:**

Open a new Claude Code session and type:

```
/morning-recap
```

If Slack is connected, Claude will start pulling from #deal-flow. If it fails with a Slack authentication error, go back to Settings → MCP Servers and check that the Slack integration shows as "Connected".

---

## Getting updates

When commands are updated, pull the latest from GitHub and copy them across:

```bash
cd ~/Projects/project-a-claude-commands && git pull
cp commands/*.md ~/.claude/commands/
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
Pulls every new message from #deal-flow since the last Morning Recap (auto-anchored), routes all deals to the correct thesis owner, captures any Slack-explicit action items, enriches entries missing a description, and posts a Morning Recap + Affinity Check List to #automation-tests. Runs automatically every weekday at 8am Berlin time.

**Flow:** Auto-anchor → Pull Slack → Parse + route entries → Capture action items → Enrich missing descriptions → Post Morning Recap + Affinity Check List to #automation-tests

**Anchor exclusion rule:** The last Morning Recap / Daily Dealflow post by Kieran is used as the time boundary only — its text is never parsed for company or LinkedIn entries, and its thread replies are also skipped entirely. Thread replies on that anchor message are where the Net New to Affinity post now lives, so they must not be processed here.

**Affinity Check List format:** The second message posted to #automation-tests contains every entry from the Morning Recap — all thesis sections, including LinkedIn profiles — one per line, as plain URLs (not Slack `<url|text>` link syntax). Entries are not grouped by thesis and descriptions are omitted.

**Requires:** Slack MCP

---

### `/net-new-affinity`
Run this after your Affinity scraper. Paste your found/not-found results, confirm Net New entries, do the LinkedIn double-check, and get an enriched Net New to Affinity post (with descriptions and funding info) posted to #automation-tests.

**Flow:** Paste scraper results → Confirm Net New → LinkedIn double-check → Enrich descriptions + funding → Post to #automation-tests

**Requires:** Slack MCP

---

### `/deal-flow-review`
Given a team member's name and a date range, pulls all #deal-flow messages where they were tagged (action items) or where entries match their thesis (thesis matches). Formats a clean summary and sends it as a Slack DM to that person.

**Usage:** `/deal-flow-review` then enter a name when prompted and select a date range (this week / last 2 weeks / custom)

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

### `/dealflow-screener`
Paste any ad-hoc deal flow list (from an email, WhatsApp, DM, etc.) and get back two separate posts to #automation-tests.

**Part 1 — Companies:** Claude enriches every company entry (WebFetch if a URL is provided, WebSearch for exact-name matches only), routes to the correct thesis owners, and composes a clean digest. Flags entries with no exact match and asks for a URL before composing.

**Part 2 — Profiles:** LinkedIn profiles are handled separately. Profiles that already have context (inline background note) are routed directly. For profiles with no context, Claude pauses and surfaces a numbered list — you click through, write a brief note on each person, and Claude routes and composes Part 2 from your notes.

**Enrichment rule:** Only uses web results where the company name is an exact match — never assumes or guesses. If the original list includes a description, that is kept even if no URL is found.

**Section labels ignored:** All entries are screened regardless of labels like "Declined", "Portfolio - Closed", "Pipeline", etc.

**Usage:** `/dealflow-screener` then paste your list (or paste it directly in the prompt)

**Requires:** Slack MCP

---

### `/dealflow-retro-newsletter`
Bi-weekly digest of all European tech funding rounds. Import a Crunchbase Pro CSV export (filtered to Europe + date range), Claude supplements with EU-Startups WebFetch, you paste Affinity cross-reference results, and Claude generates a formatted HTML email table — ready to copy into Gmail and send to the investment team.

**Stage filter:** Only Pre-Seed, Seed, and Series A rounds are included. Series B and beyond, fund closes, and growth rounds are automatically excluded.

**Affinity checklist:** Each company is listed with its website URL (for the Affinity scraper). For seen companies, paste the Affinity organisation URL inline — `1: seen | https://projecta.affinity.co/...`. The email table includes a clickable "Affinity Entry" link for each seen company so readers can self-serve.

**Recipients:** Investment Team, Anton Waitz, Uwe Horstmann, Florian Heinemann, Thies Sander, Philipp Werner, Malin Posern, Jack Wang.

**Usage:** `/dealflow-retro-newsletter` then follow prompts (confirm date range → export Crunchbase CSV → Affinity check → email opens in browser)

**Requires:** Crunchbase Pro subscription (for CSV export). EU-Startups is fetched automatically as a supplementary source. Dealroom CSV can be provided alongside for better coverage if access is available.

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
│   ├── dealflow-retro-newsletter.md
│   └── dealflow-screener.md
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
