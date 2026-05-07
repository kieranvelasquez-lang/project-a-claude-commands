---
name: Affinity contact check — correct MCP endpoint
description: For dealflow retro "Did We See?" check via Affinity MCP, use get_single_list_entry with relationship-intelligence fields — NOT get_meetings_for_entity or get_notes_for_entity
type: feedback
originSessionId: 6f7886f1-bd1c-4885-879d-91c35f88f134
---

Use `get_single_list_entry(list_id=99030, list_entry_id=X, field_types=['relationship-intelligence'])` to check for real contact in Affinity. Check the `last-contact` field — if non-null and within 12 months → "in comms 12mo".

**Why:** `get_notes_for_entity` returns manually typed meeting summaries — not proof of contact. `get_meetings_for_entity` only captures calendar events and misses email-only contact entirely (e.g., Performativ had email contact in Sept 2025 but zero calendar events). Affinity's "Losing Touch" / "Never Contacted" badges are driven by the `last-contact` relationship-intelligence field, which captures both emails and meetings.

**How to apply:** Three-step MCP process in Steps 6–7 of dealflow-retro-newsletter:
1. `search_companies(term=[Name])` → get `company_id`
2. `get_company_list_entries(company_id)` → filter for `listId == 99030` → get `list_entry_id`; "Resource not found" means company exists in Affinity but has no list entries (still show no link)
3. `get_single_list_entry(list_id=99030, list_entry_id=X, field_types=['relationship-intelligence'])` → check `last-contact` date is non-null and within 12 months

**Affinity link rule:** Show "View in Affinity" link for ALL companies found in Affinity, regardless of contact status. "—" only for companies with no Affinity presence at all.

**Label:** "in comms 12mo" (not "Yes") for contacted companies.
