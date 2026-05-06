# desk-monkey-contact-migration

**One-shot Notion → Attio migration.** Pulls all CRM data from Notion (Contacts, Deals, Activity Log, DEFCON Tasks) and creates equivalent Attio records (People, Companies, Deals, Notes, Tasks). Idempotent — gated by an Attio Note marker on a designated migration tracker record.

After successful migration, Notion CRM DBs are archived (manual operator step in Notion UI).

## Preconditions

Before running:

1. **Liam has set up the Attio workspace** with:
   - Default People, Companies, Deals objects (these come standard)
   - Custom Deal attributes per CLAUDE.md "Required Attio Deal custom attributes" section: Stage, Buyer Behavior Stage, Identified Pain, Champion, Champion Verdict, Champion Tests Passed, Champion Test Evidence, Compelling Event, Cost of Doing Nothing, Decision Process, Decision Criteria, Open Questions, Future State, Current State, Why Change, Why Now, Why Desk Monkey, Required Capabilities, Buyer-Owned Action Ratio, Forecast Category, Service Line, Source, Deal Type, Personal Stakes, Internal Priority, Show-Stoppers, Deal Evolution.
   - Custom Person attributes (optional): Committee Role, Buying Influence, Champion Test Status, Personal Stakes.
   - A "Migration Tracker" Person Record (or Company Record — pick one, mark it as the migration anchor).

2. **Attio MCP authenticated** (direct or via Zapier with OAuth complete).

3. **Notion still has the legacy CRM DBs accessible** (Contacts at `collection://474cee31-b5fe-45e6-906a-b8463eada553`, Deals at `collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4`, Activity Log at `collection://b4edca94-b39f-4f9d-9b29-e53cde7b71c6`, DEFCON Tasks at `collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e`).

If any precondition is unmet, abort with a clear runlog message and surface to Liam.

## Step 0 — Idempotency check + stub runlog

`mcp__Attio__list_notes` filtered to the Migration Tracker Record. If any Note title starts with `MIGRATION-COMPLETE`, exit immediately with runlog message `[SKIPPED] Migration already completed: <prior date>. Manual reset required (delete that Note in Attio).`.

Otherwise append to runlog:
```
## <ISO> — desk-monkey-contact-migration [IN_PROGRESS]
```

## Step 1 — Migrate Contacts → Attio People

`mcp__Notion__notion-fetch` on `collection://474cee31-b5fe-45e6-906a-b8463eada553` to confirm schema. Query all Notion Contacts rows.

For each Notion Contact (skip rows with empty Email):

1. Build the Attio Person attribute map:
   - name (first + last from Notion Name field)
   - email_addresses: [Notion Email]
   - phone_numbers: [Notion Phone]
   - job_title: Notion Title
   - company link: find or create Attio Company by Notion Company name (use `mcp__Attio__assert_record` for Companies — upsert by name)
   - Custom attributes: Committee Role, Buying Influence, Champion Test Status (map Notion select values to matching Attio select values; create options if missing)
   - description / notes (use create_note after creating the record): convert Notion Notes field into a Note attached to this Person.
2. `mcp__Attio__assert_record` with object_id=`people` (upsert by email_addresses). Capture the Attio Record ID.
3. Build a Notion-to-Attio mapping table in memory: `notion_contact_id → attio_person_id`. Used in subsequent steps.

Track per-contact result: created / updated / failed.

## Step 2 — Migrate Deals → Attio Deals

`mcp__Notion__notion-fetch` on `collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4`. Query all Notion Deals rows.

For each Notion Deal:

1. Build the Attio Deal attribute map by copying every Selling With field from the Notion Deal property panel:
   - name = Notion Deal title
   - Stage, Buyer Behavior Stage, Identified Pain, Champion, Champion Verdict, Champion Tests Passed, Champion Test Evidence, Compelling Event, Cost of Doing Nothing, Decision Process, Decision Criteria, Open Questions, Future State, Current State, Why Change, Why Now, Why Desk Monkey, Required Capabilities, Buyer-Owned Action Ratio, Forecast Category, Service Line, Source, Deal Type, Personal Stakes, Internal Priority, Show-Stoppers, Deal Evolution
   - value (Notion Value → Attio numeric)
   - Primary Contact: linked Person record (use the Notion-to-Attio Person mapping from Step 1)
   - Economic Buyer: linked Person record (same mapping)
   - Company: linked Company record (created/upserted in Step 1)
2. `mcp__Attio__assert_record` with object_id=`deals` (upsert by name). Capture Attio Deal ID.
3. Build mapping: `notion_deal_id → attio_deal_id`.

Track per-deal result.

## Step 3 — Migrate Activity Log → Attio Notes

`mcp__Notion__notion-fetch` on `collection://b4edca94-b39f-4f9d-9b29-e53cde7b71c6`. Query all Notion Activity Log rows.

For each Activity Log row:

1. Determine the parent Attio Record (use Notion-to-Attio Deal/Person mapping based on Notion's Deal/Contact relations).
2. Build the Attio Note:
   - parent_object: `deals` (preferred) or `people` (fallback if no Deal)
   - parent_record_id: from mapping
   - title: derive from Notion Activity field. Add source-ID prefix where possible:
     - If Notion Channel = Meeting and Notion has a Fireflies URL in Notes/Summary: `MTG-<fireflies_id> — <activity title>`
     - If Channel = Email and a Gmail thread ID is identifiable: `EMAIL-<thread_id> — <activity title>`
     - Else: `LEGACY-<notion_page_id_short> — <activity title>`
   - format: `markdown`
   - content: structured body with `[Channel: ...] [Direction: ...] [Status: ...] [Source: legacy Notion Activity Log]` header, then Summary, Action Items, Raw Content, plus a final line `[migrated from Notion <YYYY-MM-DD>]`
   - created_at: Notion Timestamp (preserves chronology)
3. `mcp__Attio__create_note`. Capture Note IDs in case needed for later linking.

Track per-row result.

## Step 4 — Migrate DEFCON Tasks → Attio Tasks

`mcp__Notion__notion-fetch` on `collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e`. Query all DEFCON Tasks rows.

For each Notion Task:

1. Determine linked Attio Records (Deal + Contact via mapping).
2. Build the Attio Task content with title prefix encoding (since Attio Tasks don't support custom attributes):
   - content: `[DEFCON <1-5>] [<Category>] [<Owner: Liam | counterpart name | empty>] <Notion Task title>` followed by a blank line and Notion Notes
   - deadline_at: Notion Due
   - assignees: [liam@deskmonkeyai.com] if Notion Owner = Liam, else empty list (counterpart-owned)
   - is_completed: true if Notion Status = Done, false otherwise. (Killed → is_completed=true with content prefix `[KILLED]`. Blocked → is_completed=false with `[BLOCKED]` in content.)
   - linked_records: [Deal Record, Contact Record] from mapping
3. `mcp__Attio__create_task`.

Track per-task result.

## Step 5 — Mark migration complete

`mcp__Attio__create_note` on the Migration Tracker Record:
- title: `MIGRATION-COMPLETE-<ISO date>`
- format: `markdown`
- content: `Notion → Attio migration complete on <ISO>. Stats: <N> Contacts, <M> Deals, <K> Notes, <L> Tasks migrated.\n\nLegacy Notion DBs to archive in Notion UI:\n- Contacts (collection://474cee31-b5fe-45e6-906a-b8463eada553)\n- Deals (collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4)\n- Activity Log (collection://b4edca94-b39f-4f9d-9b29-e53cde7b71c6)\n- DEFCON Tasks (collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e)\n\nDo this manually: right-click each in the Notion sidebar → Delete → empty Trash.`

## Step 6 — Final runlog + commit

```
## <ISO> — desk-monkey-contact-migration
**Expected:** Notion CRM (Contacts + Deals + Activity Log + DEFCON Tasks) → Attio
**Actual:**
- Contacts: <pulled> from Notion, <created> + <updated> in Attio, <skipped> (empty email), <failed> with reasons
- Deals: <pulled>, <created>, <updated>, <failed>
- Activity Log: <pulled>, <created as Notes>, <skipped> (no Deal/Contact mapping), <failed>
- DEFCON Tasks: <pulled>, <created>, <failed>
- Migration-complete marker: created on Attio Migration Tracker Record
**Status:** ✅ Complete | ⚠️ Partial | 🔴 Failed
**Operator follow-up:** archive the legacy Notion CRM DBs (right-click → Delete in Notion UI).
```

Commit: `git add memory/runlog.md && git commit -m "contact-migration Notion→Attio <ISO>" && git push origin main`.

## Hard rules

- NEVER run twice. Step 0 Migration-Tracker Note check gates this.
- ON duplicate-record errors in Attio: assert_record handles upsert; never create dupes.
- NEVER overwrite a populated Attio attribute with empty data from Notion. Only fill blanks.
- ALWAYS preserve Notion Timestamps via Attio created_at where possible (preserves chronology).
- ALWAYS write the runlog and commit before exit.
- After migration: tell Liam to archive Notion CRM DBs manually. Don't try to delete via MCP (Notion API limits).
