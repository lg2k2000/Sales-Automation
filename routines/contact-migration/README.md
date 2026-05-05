# contact-migration

One-shot cloud routine. Pulls Notion Contacts (Desk Monkey Sales Hub Contacts sub-DB at `4398061e507048eca39e754c547f0690`) and creates Google Contacts via Zapier MCP. Runs once. Idempotent: gated by `memory/state.json.contact_migration_completed_at`.

## Trigger
- **Schedule:** none. Manual one-shot, run from the Anthropic routine UI.
- **Connectors needed:** Notion, Gmail (for label management), Zapier (with Google Contacts: Create / Find / Update enabled)
- **Routine prompt:** `Read routines/contact-migration/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## Required Zapier actions enabled
- Google Contacts: Create Contact
- Google Contacts: Find Contact
- Google Contacts: Update Contact

## What it does
1. Reads all Notion Contacts from the sub-DB
2. Skips empty-email rows
3. Ensures Gmail labels exist for each active Deal + triage state buckets
4. Creates Google Contacts via Zapier; falls back to Find + Update on duplicate-email errors
5. Marks complete in state.json so it can't run twice by accident

## Re-running
If you need to re-run (because the source list grew or migration partially failed): manually clear `contact_migration_completed_at` in `memory/state.json`, commit, then trigger the routine again.
