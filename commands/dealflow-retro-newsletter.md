---
description: Monthly European funding round retro — source, cross-reference Affinity, output email
allowed-tools: Read, Write, WebFetch, WebSearch, Bash(open:*)
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
>    - **Funding Type:** Pre-Seed, Seed, Series A
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

**Stage filter:** After merging all sources, remove any company whose stage is Series B, Series C, Series D, Growth Equity, or any round beyond Series A. Keep: Pre-Seed, Seed, Series A, Series A+, Undisclosed. Also remove VC fund closes (Fund, Fund Close) — these are not relevant. Do not include a "Source" column in the preview or the final email.

**Country filter:** After applying the stage filter, remove any company whose headquarters is **not** in the following list: Austria, Belgium, Bulgaria, Croatia, Cyprus, Czech Republic, Denmark, Estonia, Finland, France, Germany, Greece, Hungary, Ireland, Italy, Latvia, Lithuania, Luxembourg, Malta, Netherlands, Poland, Portugal, Romania, Slovakia, Slovenia, Spain, Sweden, United Kingdom, Switzerland, Norway. This explicitly excludes Turkey and any other non-listed country.

---

## Step 4 — Enrich missing fields

For any row missing a website URL or a description:
1. WebSearch `[Company Name] startup [Country]` to identify the likely official website
2. **Verify the URL before using it**: WebFetch the homepage to confirm it resolves and the content matches the company
   - If the fetch succeeds and content matches → use the URL and extract a 1-sentence description
   - If the fetch returns an error (403, 404, timeout, domain-for-sale page, or unrelated content) → do **not** embed the URL; mark the website field as `[needs review]`
3. Never embed a URL that has not been successfully verified via WebFetch

Enrich all companies. If a website or description cannot be found or verified after searching, flag that specific row with `[needs review]` in the relevant field.

---

## Step 5 — Preview structured table

Print the full structured table in the terminal:

```
[N] companies — [Start Date] – [End Date]

# | Company            | Country   | Stage    | Amount | Lead Investors        | Description                   | Did We See? | Affinity Entry
--|--------------------|-----------|---------:|--------|----------------------|-------------------------------|-------------|---------------
1 | Acme AI            | Germany   | Series A | €12M   | Sequoia, Index       | AI-powered logistics platform | (TBD)       | (TBD)
2 | Beta Labs          | France    | Seed     | €3M    | Kima Ventures        | [needs review]                | (TBD)       | (TBD)
...
```

Note: "Did We See?" and "Affinity Entry" are shown as (TBD) at preview time — they will be filled in during Steps 6–7.

Then ask:

> "Does this look correct? Any companies to add, remove, or correct before we continue? Type 'ok' to proceed or describe your changes."

Apply any corrections the user provides, then confirm the final count before moving on.

---

## Step 6 — Affinity checklist

Before outputting the checklist, look up the official website for each company (WebSearch if not already known from earlier steps). Then output a numbered list with company name and website for manual Affinity cross-reference:

```
Affinity check — please verify each company below:
https://projecta.affinity.co/lists/99030/board/views/490142-open-organizations

 1. Acme AI — https://acme.ai
 2. Beta Labs — https://betalabs.com
 3. Gamma Systems — https://gammasystems.io
 ...
[N]. Last Company — https://...

Paste results back as a simple list. For seen companies, include the Affinity link:
  1: seen | https://projecta.affinity.co/companies/[id]
  2: not seen
  3: seen | https://projecta.affinity.co/companies/[id]
  ...
```

Wait for the user to paste their results before continuing.

---

## Step 7 — Parse Affinity results

Parse each line from the user's response. Build a lookup:

| Company | Seen? | Affinity Link |
|---------|-------|---------------|
| Acme AI | Yes | https://projecta.affinity.co/companies/123 |
| Beta Labs | No | — |
| Gamma Systems | Yes | https://projecta.affinity.co/companies/456 |

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
- **Summary line**: "[N] rounds tracked. We saw [X] of them."

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

<h2>Deal Flow Retro — CW [X] | [Start Date] – [End Date]</h2>

<div class="section-label">RECIPIENTS — paste into To: field</div>
<div class="field-box">Investment Team &lt;investmentteam@project-a.vc&gt;, Anton Waitz &lt;anton.waitz@project-a.vc&gt;, Uwe Horstmann &lt;uwe.horstmann@project-a.vc&gt;, Florian Heinemann &lt;florian.heinemann@project-a.vc&gt;, Thies Sander &lt;thies.sander@project-a.vc&gt;, Philipp Werner &lt;philipp.werner@project-a.vc&gt;, Malin Posern &lt;malin.posern@project-a.vc&gt;, Jack Wang &lt;jack.wang@project-a.vc&gt;</div>

<div class="section-label">SUBJECT</div>
<div class="field-box">Deal Flow Retro — CW [X] | [DD Mon] – [DD Mon YYYY]</div>

<div class="section-label">BODY — select all text below and copy into Gmail</div>
<div class="email-body">

<p>Hi everyone,</p>

<p>Please find below this month's European funding round retro covering <strong>[Start Date] – [End Date]</strong>. [Summary line]</p>

<table style="border-collapse:collapse;width:100%;font-family:Arial,sans-serif;font-size:12px;">
  <tr>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">#</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Company</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Country</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Stage</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Amount Raised</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Lead Investors</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Description</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Did We See?</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Affinity Entry</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Source</th>
  </tr>
  <!-- INSERT ROWS HERE — one <tr> per company, all styles inline -->
</table>

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
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[<a href="[affinity link]" style="color:#1a5fa8;">View in Affinity</a> if seen, — if not seen]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;color:#888;">[Crunchbase / EU-Startups / Dealroom]</td>
</tr>
```

### Field rules

- **Company**: always linked inline to website — `<a href="..." style="color:#000;text-decoration:underline;">Name</a>`; if no website available, plain text
- **Amount**: format as "€12M" / "$5M" / "£8M" — omit the cell content (render as "—") if unknown; never write "Unknown"
- **Lead Investors**: list up to 3 names; if more, append `+ N more`; render as "—" if unknown
- **Description**: 1–2 sentence company description from enrichment; render as "—" if unavailable
- **Did We See?**: "Yes" in green (`#1a7a1a`, bold) / "No" in light red (`#c0392b`, bold)
- **Affinity Entry**: `<a href="[link]" style="color:#1a5fa8;">View in Affinity</a>` for seen companies; "—" for not seen

### Sort order

Sort the table: companies **not seen** (No) first, alphabetically by company name; then companies seen (Yes), also alphabetical. This puts new/unseen deals at the top for the team's attention.

---

## Step 10 — Open in browser

```bash
open /Users/kvelasquez/Desktop/dealflow-retro-newsletter.html
```

---

## Step 11 — Done message

Output:

```
Done. [N] companies in this retro — [X] seen, [W] not seen.

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
- Subject format: `Deal Flow Retro — CW [X] | [DD Mon] – [DD Mon YYYY]`
- CW number = ISO week number of the period's **end date**
- Country = country only, never city (strip city names from Crunchbase location strings)
- Enrich all companies — only flag `[needs review]` if a specific field genuinely cannot be found or verified after searching
- Website URLs must be verified via WebFetch before embedding — never use an unverified URL in the checklist or HTML output
- If Dealroom CSV is provided, its data takes precedence over Crunchbase on conflicting fields
- Country whitelist: EU 27 + UK + Switzerland + Norway only — Turkey and all other countries are excluded at the filter stage
- Sort order: not-seen companies first (alphabetical), then seen companies (alphabetical)
- **Did We See?** styling: "Yes" bold green (`#1a7a1a`) / "No" bold light red (`#c0392b`)
- **Affinity Entry**: "View in Affinity" hyperlink (`color:#1a5fa8`) for seen companies; "—" for not seen
- Affinity Master Deals List: https://projecta.affinity.co/lists/99030/board/views/490142-open-organizations
