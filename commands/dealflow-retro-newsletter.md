---
description: Monthly European funding round retro — source, cross-reference Affinity, output email
allowed-tools: Read, Write, WebFetch, WebSearch, Bash(open:*), mcp__claude_ai_Affinity__search_companies, mcp__claude_ai_Affinity__get_company_list_entries
---

# Deal Flow Retro Newsletter

Monthly digest of all European tech funding rounds — cross-referenced against Affinity — formatted as a ready-to-send email to the investment team and all partners.

---

## Step 1 — Confirm date range

Based on today's date, suggest the most recently completed calendar month (e.g. if today is 1 Apr 2026, suggest "1 Mar – 31 Mar 2026").

Ask the user:

> "What month should this retro cover? Suggested: **[suggested range]**. Confirm or provide a different range."

Wait for confirmation before proceeding.

---

## Step 2 — Crunchbase CSV import

Ask the user to export data from Crunchbase Pro:

> "Please export this period's European funding rounds from Crunchbase Pro:
>
> 1. Go to **Crunchbase Pro → Search → Funding Rounds**
> 2. Set filters:
>    - **Headquarters Location:** Austria, Belgium, Bulgaria, Croatia, Cyprus, Czech Republic, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, Netherlands, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden, United Kingdom, Switzerland, Norway
>    - **Announced Date:** [Start Date] to [End Date]
>    - **Funding Type:** Pre-Seed, Seed *(Series A excluded — retro targets pre-seed for seed follow-on and seed for Series A follow-on)*
> 3. Export to CSV
> 4. Paste the file path here (e.g. ~/Downloads/crunchbase-export.csv)"

Read the CSV using the Read tool. Map columns flexibly — Crunchbase column names vary but typically include:

| Crunchbase Column | Internal Field |
|---|---|
| Organization Name | Company |
| Organization Name URL | Website |
| Description / Short Description | Description |
| Funding Round Type | Stage |
| Money Raised | Amount |
| Lead Investors | Lead Investors |
| Headquarters Location | Country |
| Announced Date | Date |

If a column is missing or named differently, infer from context. Strip city names from Country (keep country only).

---

## Step 3 — Supplementary sourcing (EU-Startups)

WebFetch the EU-Startups weekly funding round-up articles covering the same period.

1. Fetch `https://www.eu-startups.com/category/funding/` to find the relevant weekly articles for the date range
2. Fetch each relevant article and parse: company name, country, amount, stage, investors, brief description
3. Compare against the Crunchbase data by company name (case-insensitive, strip legal suffixes like "GmbH", "Ltd", "SAS")
4. Add any companies NOT already present as supplementary entries — mark their source as "EU-Startups"

If the user also provides a Dealroom CSV path, read it and merge using the same deduplication logic. Dealroom data takes precedence over Crunchbase on conflicting fields.

**Stage filter:** After merging all sources, keep only Pre-Seed and Seed. Remove Series A, Series A+, Series B, Series C, Series D, Growth Equity, and any later stage. Also remove VC fund closes (Fund, Fund Close). Do not include a "Source" column in the preview or the final email.

**Country filter:** After applying the stage filter, remove any company whose headquarters is **not** in the following list: Austria, Belgium, Bulgaria, Croatia, Cyprus, Czech Republic, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, Netherlands, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden, United Kingdom, Switzerland, Norway. This explicitly excludes Turkey and any other non-listed country.

**Minimum raise filter:** After applying the country filter, remove any company where the Money Raised is a **known value below €1,000,000** (or the equivalent in GBP, USD, SEK, or any other currency). If the amount is blank, "Undisclosed", or not provided, **keep** the company — do not penalize for non-disclosure.

---

## Step 4 — Enrich missing fields

For any row missing a website URL or a description:
1. WebSearch `[Company Name] startup [Country]` to identify the likely official website
2. **Verify the URL before using it**: WebFetch the homepage to confirm it resolves and the content matches the company
   - If the fetch succeeds and content matches → use the URL and extract a **2–5 word category label** describing what the company does (e.g., `Autonomous driving technology`, `B2B payments infrastructure`, `AI-native HR software`). Do not write full sentences or include founding context, university affiliations, or marketing language.
   - If the fetch returns an error (403, 404, timeout, domain-for-sale page, or unrelated content) → do **not** embed the URL; mark the website field as `[needs review]`
3. Never embed a URL that has not been successfully verified via WebFetch

Enrich all companies. If a website or description cannot be found or verified after searching, flag that specific row with `[needs review]` in the relevant field. Descriptions must always be 2–5 word category labels — never full sentences.

---

## Step 4.5 — Route by thesis

Route every company to one of the 5 theses using the description and company domain. Apply the routing test: is this company *building* AI/software infrastructure, or *using* it for a specific domain? Route to the domain if the latter.

| What they build | Thesis |
|---|---|
| AI agents, orchestration, LLM infra, dev tools, enterprise AI-native SaaS, software/AI-first robotics | Future of Autonomous Work |
| Fintech, payments, insurance, compliance, legal, payroll, tax, blockchain, crypto, web3 | Fintech |
| Supply chain, logistics, manufacturing, materials, hardware/industrial robotics | Global Supply Chain |
| Defense, hardware, chips, non-GNSS navigation, industrial security, cloud infra | European Resilience |
| Health, biotech, edtech, consumer, gaming, fitness, energy, creator, construction, agriculture | Surf and Turf |

**Hardcoded routing rules:**
- Cybersecurity (pentesting, infosec, security tooling) → Future of Autonomous Work
- Robotics (software/AI-first: foundation models, robot OS, physical intelligence) → Future of Autonomous Work
- Robotics (hardware/industrial/applied) → Global Supply Chain
- Defense (including defense robotics) → European Resilience — overrides all other routing
- Blockchain / crypto / web3 → Fintech

If a company cannot be confidently routed after applying the table, pause and ask the user before proceeding. Do not place uncertain entries in a catch-all; flag them explicitly.

---

## Step 5 — Apply cap and preview structured table

**50-company cap:** If more than 50 companies remain after all filters, sort by amount raised descending (largest first) and keep the top 50. Companies with unknown/undisclosed amounts rank last in this sort (keep them only if there is still room within 50). Print a note before the preview:

> "[N] companies after filtering → capped to top 50 by raise amount."

Print the structured preview grouped by thesis in the fixed section order:

```
[N] companies — [Start Date] – [End Date]

=== Future of Autonomous Work ===
# | Company            | Country   | Stage     | Amount | Lead Investors        | Description                   | In Comms 12mo | Ever in Master Deals | Affinity Entry
--|--------------------|-----------|---------:|--------|----------------------|-------------------------------|---------------|----------------------|---------------
1 | Acme AI            | Germany   | Pre-Seed  | €5M    | Sequoia              | AI-powered dev tool           | (TBD)         | (TBD)                | (TBD)

=== Fintech ===
# | Company            | Country   | Stage    | Amount | Lead Investors        | Description                   | In Comms 12mo | Ever in Master Deals | Affinity Entry
2 | Beta Labs          | France    | Seed     | €12M   | Kima Ventures        | Payments infra                | (TBD)         | (TBD)                | (TBD)
...
```

Companies are numbered sequentially across all sections. Within each section, order is **Pre-Seed first, then Seed** (alphabetical within each stage). "In Comms 12mo", "Ever in Master Deals", and "Affinity Entry" are shown as (TBD) at preview time — filled in during Steps 6–7.

Only include sections that have entries. Use the fixed order: Future of Autonomous Work → Fintech → Global Supply Chain → European Resilience → Surf and Turf.

Then ask:

> "Does this look correct? Any companies to add, remove, or correct before we continue? Type 'ok' to proceed or describe your changes."

Apply any corrections the user provides, then confirm the final count before moving on.

---

## Step 6 — Automated Affinity check (MCP)

For each company in the list, run MCP lookups to populate two independent columns: **"In Comms 12mo"** and **"Ever in Master Deals"**. Process all companies in parallel batches — no user input required.

### Per-company logic

**Call 1 — Search for the company:**

Use `search_companies(term="[Company Name]")`.

- Pick the best match: exact name first, then domain match. Use the verified website domain (from Step 4) to disambiguate.
- If no results: retry with the company domain as the search term
- If still no results: mark company as `[not found in Affinity]` — both columns default to `No`

**Call 2 — Ever in Master Deals (list 99030, not time-bounded):**

Use `get_company_list_entries(company_id=[id from Call 1])`.

- If any entry has `listId: 99030` → **Ever in Master Deals = Yes**
- If no entries or `Resource not found` error → **Ever in Master Deals = No**

**Call 3 — In Comms 12mo (notes check):**

Use `get_notes_for_entity(entity_id=[id], entity_type=0)`.

- If any note has `createdAt` after (today − 365 days) → **In Comms 12mo = Yes**
- If no qualifying notes → **In Comms 12mo = No**
- Companies with `[not found in Affinity]` or global-only records that return `Resource not found` on list entries → **In Comms 12mo = No**

The 12-month cutoff = today's date minus 365 days.

### Output after completing all lookups

```
Affinity check complete — [N] companies processed.

In Comms 12mo (Yes): [X] — [company names]
Ever in Master Deals (Yes): [Y] — [company names]
Not found in Affinity: [Z] — [company names]
```

Then ask:
> "Does this look correct? Any manual corrections before I generate the HTML? Type 'ok' to continue."

Wait for the user's response before continuing.

---

## Step 7 — Apply Affinity results

Apply any manual corrections the user provides from Step 6. Then build the internal lookup used for HTML generation:

| Company | In Comms 12mo | Ever in Master Deals | Affinity ID | Affinity Link |
|---------|---------------|----------------------|-------------|---------------|
| Acme AI | Yes | Yes | 289372786 | https://www.affinity.co/companies/289372786 |
| Beta Labs | No | No | — | — |
| Gamma Systems | [not found] | [not found] | — | — |

For `[not found in Affinity]` entries: render both Affinity columns as "No" (red) and "Affinity Entry" as "—" in the HTML.

---

## Step 8 — Pre-HTML missing info check

Before generating the HTML, scan all companies for missing data. If any row is still missing a website URL, lead investors, or description (including any flagged `[needs review]`), pause and output:

> "The following companies are missing information — please provide corrections before I generate the HTML:
> - [Company]: missing [website / lead investors / description]
> - ...
>
> Paste corrections in the format: `[Company Name]: website=[url], investors=[names], description=[text]`
> Type 'skip' to generate with blanks (rendered as '—')."

Wait for the user's response and apply any corrections before proceeding.

---

## Step 9 — Generate HTML email file

Write the file to `/Users/kvelasquez/Desktop/dealflow-retro-newsletter.html` using the Write tool.

Calculate:
- **CW number**: ISO week number of the end date of the period
- **Summary line**: "[N] rounds tracked across [X] thesis areas. We saw [Y] of them."

Use this HTML structure:

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<style>
  body { font-family: Arial, sans-serif; font-size: 13px; line-height: 1.5; max-width: 1100px; margin: 40px auto; padding: 0 20px; color: #000; }
  .section-label { color: #888; font-size: 11px; margin-bottom: 4px; text-transform: uppercase; letter-spacing: 0.05em; }
  .field-box { background: #f5f5f5; padding: 10px 14px; margin-bottom: 24px; border-radius: 4px; font-family: monospace; font-size: 12px; white-space: pre-wrap; word-break: break-all; }
  .email-body { padding: 10px 0; }
  .email-body p { margin: 0 0 16px 0; }
  h2 { font-size: 15px; border-bottom: 1px solid #ddd; padding-bottom: 6px; margin-bottom: 20px; }
</style>
</head>
<body>

<h2>Deal Flow Retro Newsletter — CW [X] | [Start Date] – [End Date]</h2>

<div class="section-label">RECIPIENTS — paste into To: field</div>
<div class="field-box">Investment Team &lt;investmentteam@project-a.vc&gt;, Anton Waitz &lt;anton.waitz@project-a.vc&gt;, Uwe Horstmann &lt;uwe.horstmann@project-a.vc&gt;, Florian Heinemann &lt;florian.heinemann@project-a.vc&gt;, Thies Sander &lt;thies.sander@project-a.vc&gt;, Philipp Werner &lt;philipp.werner@project-a.vc&gt;, Malin Posern &lt;malin.posern@project-a.vc&gt;, Jack Wang &lt;jack.wang@project-a.vc&gt;</div>

<div class="section-label">SUBJECT</div>
<div class="field-box">Deal Flow Retro Newsletter — CW [X] | [DD Mon] – [DD Mon YYYY]</div>

<div class="section-label">BODY — select all text below and copy into Gmail</div>
<div class="email-body">

<p>Hi everyone,</p>

<p>Please find below this month's European funding round retro newsletter covering <strong>[Start Date] – [End Date]</strong>. [Summary line]</p>

<!-- REPEAT THIS BLOCK FOR EACH THESIS SECTION THAT HAS ENTRIES — fixed order: Future of Autonomous Work, Fintech, Global Supply Chain, European Resilience, Surf and Turf -->

<h3 style="font-family:Arial,sans-serif;font-size:13px;font-weight:bold;margin:28px 0 8px 0;padding-bottom:4px;border-bottom:2px solid #ddd;">[Thesis Name]</h3>

<table style="border-collapse:collapse;width:100%;font-family:Arial,sans-serif;font-size:12px;">
  <tr>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">#</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Company</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Country</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Stage</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Amount Raised</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Lead Investors</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Description</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">In Comms 12mo</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Ever in Master Deals</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Affinity Entry</th>
  </tr>
  <!-- INSERT ROWS FOR THIS THESIS SECTION — sort: no first, then active; within each group: Pre-Seed → Seed → Series A, then alphabetical -->
</table>

<!-- END REPEAT -->

<p>Best,<br>Kieran</p>

</div>
</body>
</html>
```

### Table row format

Each company gets one `<tr>`. All styles must be **inline** — Gmail strips `<style>` block rules on copy-paste.

**Row template:**
```html
<tr style="background:[#fff or #fafafa for alternating];">
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[#]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;"><a href="[website]" style="color:#000;text-decoration:underline;">[Company Name]</a></td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Country]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Stage]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Amount or —]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Lead Investors or —]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;color:#555;">[Description or —]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;font-weight:bold;color:[#1a7a1a if Yes, #c0392b if No];">[Yes or No]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;font-weight:bold;color:[#1a7a1a if Yes, #c0392b if No];">[Yes or No]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[<a href="[affinity link]" style="color:#1a5fa8;">View</a> if found in Affinity, — if not]</td>
</tr>
```

### Field rules

- **Company**: always linked inline to website — `<a href="..." style="color:#000;text-decoration:underline;">Name</a>`; if no website available, plain text
- **Amount**: format as "€12M" / "$5M" / "£8M" — omit the cell content (render as "—") if unknown; never write "Unknown"
- **Lead Investors**: list up to 3 names; if more, append `+ N more`; render as "—" if unknown
- **Description**: 2–5 word category label from enrichment (e.g., `Autonomous driving technology`); render as "—" if unavailable
- **In Comms 12mo**: "Yes" bold green (`#1a7a1a`) / "No" bold red (`#c0392b`). Based on Affinity notes with `createdAt` after today − 365 days. Notes OR email interactions qualify.
- **Ever in Master Deals**: "Yes" bold green (`#1a7a1a`) / "No" bold red (`#c0392b`). Based on `listId: 99030` in `get_company_list_entries` — not time-bounded.
- **Affinity Entry**: `<a href="..." style="color:#1a5fa8;">View</a>` for any company with an Affinity ID; "—" if not found in Affinity

### Sort order (within each thesis section)

Within each thesis section: **Pre-Seed rows first, then Seed rows** — alphabetical by company name within each stage. Row numbers (#) are sequential across all sections.

---

## Step 10 — Open in browser

```bash
open /Users/kvelasquez/Desktop/dealflow-retro-newsletter.html
```

---

## Step 11 — Done message

Output:

```
Done. [N] companies in this retro — [X] active (in comms 12mo), [W] not seen.

Breakdown:
  Future of Autonomous Work: [N]
  Fintech: [N]
  Global Supply Chain: [N]
  European Resilience: [N]
  Surf and Turf: [N]

To copy the email body into Gmail:
1. Select all body text in the browser (manually, from "Hi everyone" to "Kieran")
2. Cmd+C to copy
3. Click into Gmail compose body → Cmd+V to paste (NOT Cmd+Shift+V — preserves table formatting)

Recipients and subject are shown above the body — copy those separately into the To: and Subject: fields.
```

---

## Key rules

- Never write "Unknown" anywhere — omit values or use "—"
- All table styles must be inline (Gmail strips `<style>` block rules on copy-paste)
- Subject format: `Deal Flow Retro Newsletter — CW [X] | [DD Mon] – [DD Mon YYYY]`
- CW number = ISO week number of the period's **end date**
- Country = country only, never city (strip city names from Crunchbase location strings)
- Enrich all companies — only flag `[needs review]` if a specific field genuinely cannot be found or verified after searching. Descriptions must be 2–5 word category labels, never full sentences.
- Website URLs must be verified via WebFetch before embedding — never use an unverified URL in the HTML output
- If Dealroom CSV is provided, its data takes precedence over Crunchbase on conflicting fields
- Country whitelist: EU 27 + UK + Switzerland + Norway only — Turkey and all other countries are excluded at the filter stage
- **Minimum raise filter:** Remove companies with a known raise below €1M equivalent. Keep undisclosed/blank amounts.
- **50-company cap:** After all filters, if >50 remain, keep the 50 largest raises. Unknown amounts rank last in selection.
- **Thesis routing:** Route every company using the routing table in Step 4.5. Ambiguous entries pause for user confirmation before proceeding.
- **Thesis section order (fixed):** Future of Autonomous Work → Fintech → Global Supply Chain → European Resilience → Surf and Turf. Only include sections with entries.
- **Sort order within each thesis section:** Pre-Seed rows first, then Seed rows. Alphabetical within each stage. Row numbers are sequential across all sections.
- **In Comms 12mo**: "Yes" bold green (`#1a7a1a`) / "No" bold red (`#c0392b`). Check `get_notes_for_entity` — any note with `createdAt` > today − 365 days. `[not found in Affinity]` → No.
- **Ever in Master Deals**: "Yes" bold green / "No" bold red. Check `get_company_list_entries` for `listId: 99030` — no date filter. `[not found in Affinity]` or `Resource not found` → No.
- **Affinity Entry**: "View" hyperlink for any company with an Affinity ID; "—" if not found at all.
- **Affinity MCP — Step 6 lookup:** `search_companies` → `get_company_list_entries` (Master Deals) + `get_notes_for_entity` (comms check). Fallback: retry with domain. If both fail: mark `[not found in Affinity]`.
- Affinity Master Deals List ID: `99030` (URL: https://projecta.affinity.co/lists/99030/board/views/490142-open-organizations)
