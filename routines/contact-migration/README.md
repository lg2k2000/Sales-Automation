# contact-migration

**One-shot Notion → Attio migration.** Pulls all CRM data from Notion (Contacts, Deals, Activity Log, DEFCON Tasks) and creates equivalent Attio records (People, Companies, Deals, Notes, Tasks). Idempotent — gated by a Migration Tracker Note marker on a designated Attio record.

## Trigger
- **Schedule:** none. Manual one-shot, run from the Anthropic routine UI.
- **Connectors needed:** Notion (read legacy CRM DBs), Attio (write target — direct hosted MCP at `https://mcp.attio.com/mcp`).
- **Routine prompt:** `Read routines/contact-migration/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## Preconditions

Before running:

1. Liam has set up the Attio workspace with default People/Companies/Deals objects.
2. Custom Deal attributes created (per CLAUDE.md "Required Attio Deal custom attributes" — Stage, Buyer Behavior Stage, Identified Pain, Champion, Champion Verdict, Champion Tests Passed, Champion Test Evidence, Compelling Event, Cost of Doing Nothing, Decision Process, etc.).
3. A "Migration Tracker" placeholder Person Record (or Company Record) created in Attio. The migration adds a Note with title `MIGRATION-COMPLETE-<date>` to this record on success.
4. Attio hosted MCP authenticated in Claude AND included in this routine's connector list. Run `connector-diagnostic` before migration to confirm the routine sees the required Attio capabilities.
5. Notion legacy CRM DBs still accessible (Contacts, Deals, Activity Log, DEFCON Tasks).

## What it does

1. Notion Contacts → Attio People (upsert-record by email when available; otherwise search → create / update; Companies resolved by domain first then name)
2. Notion Deals → Attio Deals with all Selling With custom attributes preserved (after `list-attribute-definitions` confirms schema; matched by Company + name + active Stage, not by name alone)
3. Notion Activity Log rows → Attio Notes attached to relevant Deal Records (title-prefixed `LEGACY-<page_id>`, `MTG-<fireflies_id>`, `EMAIL-<thread_id>` where source IDs are recoverable)
4. Notion DEFCON Tasks → Attio Tasks (title-prefix encoding `[DEFCON <N>] [<Category>] [<Owner>]`)
5. Marks Attio Migration Tracker Record with completion Note
6. Operator follow-up: archive Notion CRM DBs in Notion UI (right-click → Delete → empty Trash)

If a needed Attio capability is missing in the routine runtime, the migration logs `BLOCKED_TOOL_GAP` and stops at the affected step rather than partially running silently.

## Re-running

If you need to re-run (because the source list grew or migration partially failed):
- Delete the `MIGRATION-COMPLETE-<date>` Note on the Migration Tracker Record in Attio
- Trigger the routine again

## See also
- `prompt.md` — exact steps
- `../../CLAUDE.md` — working memory + Attio data model
- `../../decisions.md` — why Attio became canonical
