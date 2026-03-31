---
type: feedback
name: Affinity checklist format for dealflow retro
created: 2026-03-31
applies_to: dealflow-retro-newsletter (Step 6)
---

## Rule

The Affinity checklist output should be a simple seen/not seen list only. No Affinity links needed in the response.

- Format each entry as: `1: seen` or `1: not seen`
- Include the company website URL in the checklist (helps the Affinity scraper)
- "Not seen" requires no action
- For "seen" companies, ask for pass reasons separately after receiving the full list

## Why

User manually looks up pass reasons in Affinity for "seen" companies. The scraper uses website URLs to match records. Affinity links in the response are unnecessary noise.

## How to apply

In Step 6 of `dealflow-retro-newsletter`:

1. Output the checklist in the format `<number>: seen` or `<number>: not seen`, including the company website URL alongside each entry
2. Do not include Affinity links
3. After receiving the full checklist results from the user, ask for pass reasons only for the "seen" companies
