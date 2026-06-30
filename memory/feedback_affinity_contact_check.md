---
name: Affinity contact check — correct MCP endpoint
description: For dealflow retro "Did We See?" check via Affinity MCP, use get_single_list_entry with relationship-intelligence fields — NOT get_meetings_for_entity or get_notes_for_entity
type: feedback
originSessionId: 6f7886f1-bd1c-4885-879d-91c35f88f134
---

**Seen?** = company present in Master Deals List (list 99030). Being in the list is sufficient — no contact check needed. Many companies in list 99030 have null last-contact because Affinity sync is imperfect; they still count as Seen.

**In Contact (12mo)?** = an email, meeting, or formal interaction was synced in Affinity (`last-contact` or `last-interaction` non-null) within the last 12 months. Use `get_single_list_entry(list_id=99030, list_entry_id=X, field_types=['relationship-intelligence'])` to retrieve this.

**Why:** Earlier logic required non-null contact to mark Seen=Yes, causing many false negatives for companies that were clearly reviewed and added to the Master Deals List but had no synced interaction record (LinkedIn, phone, unsynced email, etc.).

**How to apply:** Three-step MCP process in Steps 6–7 of dealflow-retro-newsletter:
1. `search_companies(term=[Name])` → get `company_id`
2. `get_company_list_entries(company_id)` → filter for `listId == 99030` → get `list_entry_id`; if found → Seen=Yes automatically
3. `get_single_list_entry(list_id=99030, list_entry_id=X, field_types=['relationship-intelligence'])` → check `last-contact`/`last-interaction` date for In Contact (12mo) only

**Affinity link rule:** Show "View in Affinity" link for ALL companies found in Affinity, regardless of contact status. "—" only for companies with no Affinity presence at all.
