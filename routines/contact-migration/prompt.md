# desk-monkey-contact-migration

**One-shot Notion → Attio migration.** Pulls all CRM data from Notion (Contacts, Deals, Activity Log, DEFCON Tasks) and creates equivalent Attio records (People, Companies, Deals, Notes, Tasks). Idempotent — gated by an Attio Note marker on a designated migration tracker record.

After successful migration, Notion CRM DBs are archived (manual operator step in Notion UI).

## Step 0a — Read the Attio tool contract

Read `skills/attio-tooling.md`. Run the Attio runtime preflight before any CRM action. Append the `TOOL_CONTRACT` line to `memory/runlog.md`.

This routine performs heavy CRM writes. If any of these capabilities are missing, log `BLOCKED_TOOL_GAP` and STOP at the affected step. Migration is not safe to "partially run" without operator awareness:

- search records / list records (required for dedupe)
- create record (required for People, Companies, Deals)
- update record (required so we don't overwrite populated fields)
- list-attribute-definitions for object `deals` (required before any Deal write — needed to read actual schema and allowed select options)
- create-note (required for Activity Log → Notes and the migration-complete marker)
- create-task (required for DEFCON Tasks → Tasks)

`upsert-record` is preferred when present; if absent, fall back to search-then-create-or-update.

Browser automation is forbidden. Do NOT hardcode MCP function names — use the connector tools the runtime actually exposes.

## Preconditions

Before running:

1. **Liam has set up the Attio workspace** with:
   - Default People, Companies, Deals objects (these come standard).
   - Custom Deal attributes per CLAUDE.md "Required Attio Deal custom attributes" section.
   - Custom Person attributes (optional): Committee Role, Buying Influence, Champion Test Status, Personal Stakes.
   - A "Migration Tracker" Person Record (or Company Record — pick one, mark it as the migration anchor).

2. **Attio hosted MCP authenticated in Claude and included in the routine.** URL: `https://mcp.attio.com/mcp`. Run `connector-diagnostic` before this migration. Do not start migration if `connector-diagnostic` reported missing capabilities.

3. **Notion still has the legacy CRM DBs accessible** (Contacts at `collection://474cee31-b5fe-45e6-906a-b8463eada553`, Deals at `collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4`, Activity Log at `collection://b4edca94-b39f-4f9d-9b29-e53cde7b71c6`, DEFCON Tasks at `collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e`).

If any precondition is unmet, abort with a clear runlog message and surface to Liam.

## Step 0b — Idempotency check + stub runlog

Use the Attio list-notes / list-records capability filtered to the Migration Tracker Record. If any Note title starts with `MIGRATION-COMPLETE`, exit immediately with runlog message `[SKIPPED] Migration already completed: <prior date>. Manual reset required (delete that Note in Attio).`.

Otherwise append to runlog:
```
## <ISO> — desk-monkey-contact-migration [IN_PROGRESS]
```

## Step 1 — Migrate Contacts → Attio People

`mcp__Notion__notion-fetch` on `collection://474cee31-b5fe-45e6-906a-b8463eada553` to confirm schema. Query all Notion Contacts rows.

For each Notion Contact (skip rows with empty Email):

1. Resolve the Company first:
   - Search Attio Companies via the search-records capability by domain (preferred) and name as fallback.
   - If no match: prefer the upsert-record capability for object `companies`. If upsert-record is unavailable, use create-record for object `companies`. Match key for upsert is domain when present, name otherwise.
   - If match exists: use update-record to fill blanks only. Never overwrite populated fields.

2. Resolve the Person:
   - Search Attio People via the search-records capability by email_addresses.
   - Build the Person attribute map:
     - name (first + last from Notion Name field)
     - email_addresses: [Notion Email]
     - phone_numbers: [Notion Phone]
     - job_title: Notion Title
     - company link: the Company record from above
     - Custom attributes: Committee Role, Buying Influence, Champion Test Status, Personal Stakes — only if those custom attributes exist in the Attio workspace and the Notion select value maps to a valid Attio option.
   - For select fields: if a Notion value has no matching Attio option in the workspace, omit that field on this record and log `FIELD_OPTION_GAP: people.<field>=<notion_value>` to the runlog.
   - If no Person match: prefer upsert-record (object `people`, match key email_addresses). If upsert-record unavailable, use create-record.
   - If match exists: use update-record to fill blanks only. Never overwrite populated fields.
   - For Notion's free-text Notes field on a Contact: use create-note to attach a Note to the Person Record with title `LEGACY-<notion_page_id_short> — contact notes`.

3. Build a Notion-to-Attio mapping table in memory: `notion_contact_id → attio_person_id`. Used in subsequent steps.

Track per-contact result: created / updated / skipped (no email or no match strategy) / failed.

## Step 2 — Migrate Deals → Attio Deals

Before writing any Deal: call `list-attribute-definitions` for object `deals`. Capture the actual attribute slugs and the allowed select options for Stage, Forecast Category, Service Line, Source, Deal Type, Buyer Behavior Stage, Champion Verdict.

`mcp__Notion__notion-fetch` on `collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4`. Query all Notion Deals rows.

For each Notion Deal:

1. Build the Attio Deal attribute map. Only write fields that exist in the Attio workspace. For select fields, only write values that match an allowed option. If a Notion value has no matching Attio option, skip that field and log `FIELD_OPTION_GAP: deals.<field>=<notion_value>`.

   Candidate fields (write only those present in the workspace schema and only with valid values):
   - name = Notion Deal title
   - Stage, Buyer Behavior Stage, Identified Pain, Champion, Champion Verdict, Champion Tests Passed, Champion Test Evidence, Compelling Event, Cost of Doing Nothing, Decision Process, Decision Criteria, Open Questions, Future State, Current State, Why Change, Why Now, Why Desk Monkey, Required Capabilities, Buyer-Owned Action Ratio, Forecast Category, Service Line, Source, Deal Type, Personal Stakes, Internal Priority, Show-Stoppers, Deal Evolution
   - value (Notion Value → Attio numeric)
   - Primary Contact: linked Person record (use the Notion-to-Attio Person mapping from Step 1)
   - Economic Buyer: linked Person record (same mapping)
   - Company: linked Company record (created/upserted in Step 1)

2. Match against existing Deals:
   - Search Attio Deals via the search-records capability for: same Company + same Deal name + active Stage. Do not blindly upsert by name only — multiple Deals can share a name across Companies.
   - If a likely-same Deal exists: use update-record to fill blanks only. Never overwrite populated fields.
   - If no match: prefer upsert-record for object `deals` only if a reliable unique matching attribute exists in the workspace. Otherwise use create-record for object `deals`.

3. Capture Attio Deal ID. Build mapping: `notion_deal_id → attio_deal_id`.

4. If a relevant sales pipeline list exists in Attio AND the list-lists / add-record-to-list capabilities are available:
   - Call `list-list-attribute-definitions` for the target list to discover required entry attributes.
   - Use `add-record-to-list` with the Deal record ID and any required entry attributes.
   - If list-lists or add-record-to-list is unavailable: log `BLOCKED_TOOL_GAP: Attio list membership unavailable` and skip list assignment. The Deal record still exists.

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
   - created_at: Notion Timestamp (preserves chronology) if create-note exposes the field
3. Use the create-note capability. Capture Note IDs in case needed for later linking.

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
3. Use the create-task capability.

Track per-task result.

## Step 5 — Mark migration complete

Use the create-note capability on the Migration Tracker Record:
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
- FIELD_OPTION_GAP entries: <count> (see runlog detail)
- BLOCKED_TOOL_GAP entries: <count>
- Migration-complete marker: created on Attio Migration Tracker Record
**Status:** ✅ Complete | ⚠️ Partial | 🔴 Failed
**Operator follow-up:** archive the legacy Notion CRM DBs (right-click → Delete in Notion UI). For any FIELD_OPTION_GAP, decide whether to add the missing Attio select options and re-run the affected rows.
```

Commit: `git add memory/runlog.md && git commit -m "contact-migration Notion→Attio <ISO>" && git push origin main`.

## Hard rules

- NEVER run twice. Step 0b Migration-Tracker Note check gates this.
- NEVER use `assert_record` or any other fake tool name. Use upsert-record if present, otherwise search → create / update.
- NEVER write Deal fields without first calling `list-attribute-definitions` for object `deals`.
- NEVER overwrite a populated Attio attribute with empty data from Notion. Only fill blanks.
- NEVER write a select value that isn't an allowed option in the workspace. Log `FIELD_OPTION_GAP` and skip the field.
- NEVER blindly upsert Deals by name only. Match by Company + name + active Stage.
- NEVER use browser automation.
- ALWAYS preserve Notion Timestamps via Attio created_at where possible (preserves chronology).
- ALWAYS run `connector-diagnostic` before this migration.
- ALWAYS write the runlog and commit before exit.
- After migration: tell Liam to archive Notion CRM DBs manually. Don't try to delete via MCP (Notion API limits).
