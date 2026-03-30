---
description: Bi-weekly European funding round retro — source, cross-reference Affinity, capture pass reasons, output email
allowed-tools: Read, Write, WebFetch, WebSearch, Bash(open:*)
---

# Deal Flow Retro Newsletter

Bi-weekly digest of all European tech funding rounds — cross-referenced against Affinity, with pass reasons captured — formatted as a ready-to-send email to the investment team and all partners.

---

## Step 1 — Confirm date range

Based on today's date, suggest the most recent completed two-week window (e.g. if today is 30 Mar 2026, suggest "17 Mar – 30 Mar 2026").

Ask the user:

> "What two-week period should this retro cover? Suggested: **[suggested range]**. Confirm or provide a different range."

Wait for confirmation before proceeding.

---

## Step 2 — Crunchbase CSV import

Ask the user to export data from Crunchbase Pro:

> "Please export this period's European funding rounds from Crunchbase Pro:
>
> 1. Go to **Crunchbase Pro → Search → Funding Rounds**
> 2. Set filters:
>    - **Headquarters Location:** Europe (or select specific countries)
>    - **Announced Date:** [Start Date] to [End Date]
>    - **Funding Type:** Seed, Series A, Series B, Series C (or leave open for all stages)
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

---

## Step 4 — Enrich missing fields

For any row missing a website URL or a description:
- WebSearch `[Company Name] startup [Country]` to find the official website
- WebFetch the homepage to extract a 1-sentence description

**Cap at 10 enrichments** to keep runtime manageable. Flag remaining unenriched rows with `[needs review]` in the Description column.

---

## Step 5 — Preview structured table

Print the full structured table in the terminal:

```
[N] companies — [Start Date] – [End Date]

# | Company            | Country   | Stage    | Amount | Lead Investors        | Source
--|--------------------|-----------|---------:|--------|----------------------|----------
1 | Acme AI            | Germany   | Series A | €12M   | Sequoia, Index       | Crunchbase
2 | Beta Labs          | France    | Seed     | €3M    | Kima Ventures        | EU-Startups
...
```

Then ask:

> "Does this look correct? Any companies to add, remove, or correct before we continue? Type 'ok' to proceed or describe your changes."

Apply any corrections the user provides, then confirm the final count before moving on.

---

## Step 6 — Affinity checklist

Output a numbered list of all company names for manual Affinity cross-reference:

```
Affinity check — please verify each company below using your Affinity Chrome shortcut or the Master Deals List:
https://projecta.affinity.co/lists/99030/board/views/490142-open-organizations

 1. Acme AI
 2. Beta Labs
 3. Gamma Systems
 ...
[N]. Last Company

Paste results back in this format (one per line):
  1: seen | https://projecta.affinity.co/organizations/12345
  2: not seen
  3: seen | https://projecta.affinity.co/organizations/67890 | passed

For each number: 'seen | [affinity link]' if we've engaged with them (append '| passed' if we passed), or 'not seen' if they're new to us.
```

Wait for the user to paste their results before continuing.

---

## Step 7 — Parse Affinity results

Parse each line from the user's response. Build a lookup:

| Company | Seen? | Affinity Link | Passed? |
|---------|-------|---------------|---------|
| Acme AI | Yes | https://... | No → "In Consideration" |
| Beta Labs | No | — | — |
| Gamma Systems | Yes | https://... | Yes → needs pass reason |

---

## Step 8 — Collect pass reasons

For every company marked `seen | ... | passed`, ask:

> "Pass reasons needed for [N] companies:
> - Gamma Systems
> - Delta Corp
> - ...
>
> Paste one per line in the format: `[Company Name]: [reason]`
> Type 'skip' to leave pass reasons blank."

Map each reason back to the corresponding company. Accept free text as-is — do not rephrase or summarise.

If the user types 'skip', leave the Pass Reason column blank (render as "—") for all passed companies.

---

## Step 9 — Generate HTML email file

Write the file to `/Users/kvelasquez/Desktop/dealflow-retro-newsletter.html` using the Write tool.

Calculate:
- **CW number**: ISO week number of the end date of the period
- **Summary line**: "[N] rounds tracked. We saw [X] of them ([Y] passed, [Z] in consideration)."

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
<div class="field-box">Investment Team &lt;investmentteam@project-a.vc&gt;, Anton Waitz &lt;anton.waitz@project-a.vc&gt;, Uwe Horstmann &lt;uwe.horstmann@project-a.vc&gt;, Florian Heinemann &lt;florian.heinemann@project-a.vc&gt;, Thies Sander &lt;thies.sander@project-a.vc&gt;, Philipp Werner &lt;philipp.werner@project-a.vc&gt;, Malin Posern &lt;malin.posern@project-a.vc&gt;, Jack Wang &lt;jack.wang@project-a.vc&gt;, Vincent Synde &lt;vincent.synde@project-a.vc&gt;, Miriam Ayasse &lt;miriam.ayasse@project-a.vc&gt;, Martin Laudien &lt;martin.laudien@project-a.vc&gt;, Christian Kurz &lt;christian.kurz@project-a.vc&gt;, Andreas Kühnke &lt;andreas.kuehnke@project-a.vc&gt;, Anton Grabovski &lt;anton.grabovski@project-a.vc&gt;, Jan-Willem Jensen &lt;jan-willem.jensen@project-a.vc&gt;, Elias Wahl &lt;elias.wahl@project-a.vc&gt;</div>

<div class="section-label">SUBJECT</div>
<div class="field-box">Deal Flow Retro — CW [X] | [DD Mon] – [DD Mon YYYY]</div>

<div class="section-label">BODY — select all text below and copy into Gmail</div>
<div class="email-body">

<p>Hi everyone,</p>

<p>Please find below this fortnight's European funding round retro covering <strong>[Start Date] – [End Date]</strong>. [Summary line]</p>

<table style="border-collapse:collapse;width:100%;font-family:Arial,sans-serif;font-size:12px;">
  <tr>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">#</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Company</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Country</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Stage</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Amount Raised</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Lead Investors</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Did We See?</th>
    <th style="background:#f0f0f0;text-align:left;padding:8px 10px;border:1px solid #ddd;">Pass Reason</th>
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
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;">[Lead Investors]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;font-weight:[bold if Yes];color:[#1a7a1a if Yes, #888888 if No];">[Yes or No]</td>
  <td style="padding:7px 10px;border:1px solid #ddd;vertical-align:top;color:#555;font-style:[italic if pass reason];">[Pass Reason or In Consideration or —]</td>
</tr>
```

### Field rules

- **Company**: always linked inline to website — `<a href="..." style="color:#000;text-decoration:underline;">Name</a>`; if no website available, plain text
- **Amount**: format as "€12M" / "$5M" / "£8M" — omit the cell content (render as "—") if unknown; never write "Unknown"
- **Lead Investors**: list up to 3 names; if more, append `+ N more`
- **Did We See?**: "Yes" in green (`#1a7a1a`, bold) / "No" in grey (`#888888`)
- **Pass Reason**: pass reason text if passed; "In Consideration" (plain) if seen but not passed; "—" if not seen

### Sort order

Sort the table: companies we've seen (Yes) first, sorted alphabetically by company name within each group; then companies we haven't seen (No), also alphabetical.

---

## Step 10 — Open in browser

```bash
open /Users/kvelasquez/Desktop/dealflow-retro-newsletter.html
```

---

## Step 11 — Done message

Output:

```
Done. [N] companies in this retro — [X] seen ([Y] passed, [Z] in consideration), [W] not seen.

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
- Enrichment via WebFetch/WebSearch is capped at 10 companies — flag the rest as `[needs review]`
- If Dealroom CSV is provided, its data takes precedence over Crunchbase on conflicting fields
- EU-Startups source flag is for internal tracking during the session only — do not include a "Source" column in the final email
- Affinity Master Deals List: https://projecta.affinity.co/lists/99030/board/views/490142-open-organizations
