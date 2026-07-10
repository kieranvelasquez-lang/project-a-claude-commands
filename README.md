# Project A — Claude Commands

Custom Claude Code slash commands for the Project A investment team.

---

## Commands

| Command | What it does |
|---|---|
| `/morning-recap` | Pulls new #deal-flow Slack messages, routes by deep dive, and posts the Morning Recap to #automation-tests |
| `/deal-flow-review` | Pulls deal flow for a specific team member + date range and sends a formatted DM to their Slack |
| `/investment-team-dealflow-meetings` | Formats a Granola meeting transcript into ready-to-send Gmail email summaries |
| `/evertrace-signals` | Reads an Evertrace CSV export, routes companies by deep dive, and posts a signal digest to Slack |
| `/dealflow-retro-newsletter` | Monthly European Pre-Seed + Seed funding round retro — imports Crunchbase CSV, manual top-50 curation, thesis routing, two-column Affinity MCP check (Seen? / In Contact 12mo?), outputs HTML email in 6 thesis tables |
| `/dealflow-screener` | Paste any ad-hoc deal flow list — Claude enriches, routes companies and profiles by deep dive, and posts a unified digest to #automation-tests |
| `/call-prep` | Give Claude a company name, website, and/or pitch deck/memo — get back a concise VC investment brief (tech, team, traction, key questions) that opens as an HTML page in your browser |

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

Open a new Claude Code session. You should now see `/morning-recap`, `/deal-flow-review`, `/investment-team-dealflow-meetings`, `/evertrace-signals`, `/dealflow-retro-newsletter`, `/dealflow-screener`, and `/call-prep` in the slash command menu.

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

## Deep Dives 2.0 — Routing

All commands use the same deep dive routing table (updated July 2026):

| What they build | Deep Dive |
|---|---|
| Space (civilian/non-defense), ocean, land, subsurface, agriculture, infrastructure, construction, energy (hardware and software for the physical world), new materials | **Physical World Intelligence** (Daria) |
| Manufacturing, factory automation, factory software, supply chain, logistics | **Industrial Autonomy** (Oskar) |
| Fintech, payments, healthcare, real estate, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | **Regulated Industries** (Marjorie) |
| Defense tech, military, weapons, military space | **European Resilience** (Jack, Miha) |
| Semiconductors, chips, quantum computing, novel computation; breakthrough energy hardware (novel generation/storage methods); frontier biotech (synthetic biology, genomics, drug discovery); novel AI architectures/paradigms, fundamental CS algorithm research, competitive-programming/algorithmic-research background (e.g. "ex-competitive programmer", "AI research background", "PhD, novel model work") — route here on any plausible fit, even from a terse note; robotics infrastructure | **Frontier Tech** (Jack, Omar) |
| AI agents, orchestration, LLM infra, dev tools, enterprise AI-native SaaS (applied/practitioner work with no research or competitive-programming signal — e.g. "AI engineer", "ML ops", generic "works in AI"), gaming, consumer, edtech, creator, fitness | **Miscellaneous** (no team member — visibility only) |

**Hardcoded overrides:**
- **Cybersecurity** — all cybersecurity (commercial pentesting/infosec/SOC/security tooling and offensive/defense-grade) → Frontier Tech (Omar)
- **AI sales tools** (commissions, revenue ops) → Miscellaneous
- **Blockchain / crypto / web3** → Regulated Industries
- **Energy** (software, cleantech, grid, infrastructure) → Physical World Intelligence (Daria); breakthrough energy hardware (e.g. fusion) → Frontier Tech
- **Agriculture, construction, infrastructure** → Physical World Intelligence (Daria)
- **Space** — civilian/commercial (satellites, launch, orbital infra) → Physical World Intelligence (Daria); military/defense space → European Resilience
- **Robotics** — all robotics infrastructure (software/AI-first: foundation models, physical intelligence, robot OS; hardware/physical world: space, ocean, agriculture, construction) → Frontier Tech (Omar); factory/industrial floor → Industrial Autonomy; defense → European Resilience
- **Biotech** — frontier (synthetic biology, genomics, drug discovery) → Frontier Tech (Omar); commercial healthtech/medtech/clinical → Regulated Industries
- **Semiconductors, chips, quantum computing** → Frontier Tech
- **AI/CS research or competitive-programming signal** (novel model/algorithm work, CS research roles, PhDs on new methods, competitive-programming background) → Frontier Tech, even on a terse note; applied AI product/practitioner roles with no research signal → Miscellaneous

**Section order (fixed across all skills):** Physical World Intelligence → Industrial Autonomy → Regulated Industries → European Resilience → Frontier Tech → Miscellaneous

These rules are baked into every skill. Company-specific overrides live in your local corrections memory (see below).

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
Pulls every new message from #deal-flow since the last Morning Recap (auto-anchored), routes all deals to the correct deep dive, captures any Slack-explicit action items, enriches entries missing a description, and posts the Morning Recap to #automation-tests.

**Flow:** Auto-anchor → Pull Slack → Parse + route entries → Capture action items + commentary → Enrich missing descriptions → Post Morning Recap to #automation-tests

**Inline commentary:** Every entry preserves two sources of commentary as `  › _<@person>: "note"_` sub-bullets beneath the entry line:
- **Main message notes** — any text the poster wrote around a link on the same line (e.g. "Met at NOAH: https://url", "https://url — pass, no EU traction")
- **Thread reply comments** — substantive replies from humans in the thread (pure routing phrases like "Tagging to X" are excluded; replies that mix routing + commentary are included in full)

Commentary sub-bullets appear after the main entry line, main message note first then thread replies in order. Entries with no commentary are unchanged.

**Anchor exclusion rule:** The last Morning Recap by Kieran is used as the time boundary only — its text is never parsed for company or LinkedIn entries, and its thread replies are skipped entirely.

**Requires:** Slack MCP

---

### `/deal-flow-review`
Given a team member's name and a date range, pulls all #deal-flow messages where they were tagged (action items) or where entries match their deep dive (deep dive matches). Formats a clean summary and sends it as a Slack DM to that person.

**Usage:** `/deal-flow-review` then enter a name when prompted and select a date range (this week / last 2 weeks / custom)

**Requires:** Slack MCP

---

### `/investment-team-dealflow-meetings`
Paste a Granola meeting transcript and get back two ready-to-send email summaries — Version 1 (internal, full detail) and Version 2 (clean, compliance-facing). Output opens directly in your browser for copy-paste into Gmail.

**V1 recipients:** Investment Team, Anton Waitz, Uwe Horstmann, Florian Heinemann, Thies Sander, Philipp Werner, Malin Posern, Jack Wang.

**V2 recipients (compliance):** Investment Team, Anton Waitz, Vincent Synde, Miriam Ayasse, Martin Laudien, Christian Kurz, Andreas Kühnke, Anton Grabovski, Elias Wahl, Christoph Heiland, Frieder Zinkel.

**Usage:** `/investment-team-dealflow-meetings` then paste transcript when prompted

**Requires:** Nothing beyond Claude Code

---

### `/evertrace-signals`
Export a CSV from Evertrace, run this command, answer three questions (CSV path, week label, optional theme mapping), and get a clean signal digest posted to Slack. Stealth profiles are always included and merged inline with New Company entries per deep dive. Companies are auto-routed to deep dive owners.

**Usage:** `/evertrace-signals` then follow prompts (drag CSV into terminal for path)

**Requires:** Slack MCP

---

### `/dealflow-screener`
Paste any ad-hoc deal flow list (from an email, WhatsApp, DM, etc.) and get back a unified post to #automation-tests with companies and profiles merged by deep dive.

**Companies:** Claude enriches every company entry (WebFetch if a URL is provided, WebSearch for exact-name matches only), routes to the correct deep dive, and composes a clean digest. Flags entries with no exact match and asks for a URL before composing.

**Profiles:** Profiles with an inline background note are routed directly. For profiles with no context, Claude pauses and surfaces a numbered list — you click through, write a brief note on each person, and Claude routes and includes them in the same unified message.

**Enrichment rule:** Only uses web results where the company name is an exact match — never assumes or guesses. If the original list includes a description, that is kept even if no URL is found.

**Section labels ignored:** All entries are screened regardless of labels like "Declined", "Portfolio - Closed", "Pipeline", etc.

**Usage:** `/dealflow-screener` then paste your list (or paste it directly in the prompt)

**Requires:** Slack MCP

---

### `/call-prep`
Give Claude a company name plus any available materials (website URL, pitch deck PDF path, DOCX memo, or pasted text) and get back a concise VC investment brief that opens as an HTML page in your browser — ready to read before the call.

**What the brief covers:** Snapshot · Problem & Solution · Product & Technology (plain-language explainer of underlying science/engineering for deeptech/hardware) · Traction & Metrics · Team · Market & Timing · Investment Framing · Key Questions (5 max)

**Research approach:** Reads documents first (highest signal), fetches homepage only, then runs 5 targeted web searches — optimised for low token spend.

**Output:** HTML file opened in browser (`/tmp/call-prep-[Company].html`).

**Usage:** `/call-prep` then provide company name + materials, or paste them directly in the prompt

**Requires:** Nothing beyond Claude Code

---

### `/dealflow-retro-newsletter`
Monthly digest of European Pre-Seed and Seed funding rounds, organised into six thesis tables. Import a Crunchbase Pro CSV export, Claude supplements with EU-Startups WebFetch, you manually curate the top 50 companies, Claude routes them by thesis and runs an automated Affinity MCP check, then generates a formatted HTML email ready to copy into Gmail.

**Stage filter:** Pre-Seed and Seed only. Series A and above, fund closes, and growth rounds are excluded.

**Flow:**
1. Confirm date range → export Crunchbase CSV (Pre-Seed + Seed filter)
2. EU-Startups supplementary fetch + merge + country/stage filter
3. Preview full raw list → you curate to top 50 (enrichment only runs on selected companies)
4. Enrich selected companies (WebSearch + WebFetch to verify URLs and get descriptions)
5. Thesis routing: Claude assigns each company to one of 6 thesis areas → you confirm
6. Affinity check: test batch of 5 first → confirm logic → run all 50
7. HTML email generated → opens in browser

**Affinity check (two columns):**
- **Seen?** — company is in Master Deals List (99030). Being in the list is the signal — null contact records do not mean unseen.
- **In Contact (12mo)?** — `last-contact` or `last-interaction` within the past 12 months.
- All companies found in Affinity get a "View in Affinity" link regardless of contact status.
- If a name search fails, Claude retries by domain (handles rebrands and generic names).

**Output:** Six thesis tables (Physical World Intelligence → Industrial Autonomy → Regulated Industries → European Resilience → Frontier Tech → Miscellaneous), sorted Pre-Seed before Seed, with three priority tiers per stage: unseen (No/No) → seen not contacted (Yes/No) → active relationship (Yes/Yes).

**Recipients:** Investment Team, Anton Waitz, Uwe Horstmann, Florian Heinemann, Thies Sander, Philipp Werner, Malin Posern, Jack Wang.

**Usage:** `/dealflow-retro-newsletter` then follow prompts

**Requires:** Crunchbase Pro subscription (CSV export). Affinity MCP must be connected. Dealroom CSV can be provided alongside Crunchbase for better coverage.

---

## Repo structure

```
project-a-claude-commands/
├── README.md
├── commands/
│   ├── morning-recap.md
│   ├── deal-flow-review.md
│   ├── evertrace-signals.md
│   ├── investment-team-dealflow-meetings.md
│   ├── dealflow-retro-newsletter.md
│   ├── dealflow-screener.md
│   └── call-prep.md
└── memory/
    ├── morning-recap/                       ← corrections + feedback for /morning-recap
    ├── deal-flow-review/                    ← (created when first correction is learned)
    ├── evertrace-signals/                   ← (created when first correction is learned)
    ├── investment-team-dealflow-meetings/   ← (created when first correction is learned)
    └── dealflow-retro-newsletter/           ← (created when first correction is learned)
```

Each skill gets its own subfolder under `memory/` as it accumulates corrections and feedback over time. Only `morning-recap/` is populated today — the others are created automatically the first time a correction is learned for that skill.

---

## Questions / issues

Ping Kieran on Slack.
