# desk-monkey-contact-migration

One-shot routine. Pulls Notion Contacts from the Desk Monkey Sales Hub Contacts sub-DB and creates Google Contacts via Zapier. Tags each in Gmail with the matching Deal label. Idempotent via `state.json`.

## Step 0 — Sync repo + stub runlog
- Read `memory/state.json`. If `contact_migration_completed_at` is set, exit immediately and write a short runlog entry: "Migration already completed at <timestamp>. Manual reset required to re-run."
- Otherwise append:
  ```
  ## <ISO> — desk-monkey-contact-migration [IN_PROGRESS]
  ```

## Step 1 — Pull Notion Contacts
`mcp__Notion__notion-query-database-view` on collection `4398061e507048eca39e754c547f0690` (Desk Monkey Contacts sub-DB). Pull all rows, no filter.

For each contact, extract:
- Full name
- Email (primary)
- Phone (if present)
- Company
- Title (if present)
- Linked Deal (relation to Deals collection)
- Notes (truncate to 200 chars)

Skip contacts where:
- Email is empty (can't create a Google Contact without it)
- Email is on @hpe.com or @anthropicidentity.com (internal, not a prospect)

## Step 2 — Ensure Gmail labels exist
Call `mcp__Gmail__list_labels`. Check for these:
- `Anthropic-Identity`
- `Authonomy`
- `Houston-Foam`
- `Blue-Ally`
- `MyVP`
- `_Inbox/Action`
- `_Inbox/Waiting`

For any missing, call `mcp__Gmail__create_label`. Skip ones that already exist.

## Step 3 — Create Google Contacts via Zapier
For each Notion contact:

- Try `mcp__Zapier__execute_zapier_write_action` with the `Google Contacts: Create Contact` action enabled. Pass: name, email, phone, company, title, notes.
- If creation returns a duplicate-email error: call `Google Contacts: Find Contact` by email, then `Google Contacts: Update Contact` to fill any missing fields without overwriting existing populated ones.
- Track results: created, updated, failed (with reason).

## Step 4 — Mark migration complete
Update `memory/state.json`:
- `contact_migration_completed_at` = ISO timestamp
- `contact_migration_stats` = `{ "pulled": N, "created": M, "updated": K, "skipped": L, "failed": F }`

## Step 5 — Final runlog + commit
Replace `[IN_PROGRESS]` with:
```
## <ISO> — desk-monkey-contact-migration
**Expected:** all eligible Notion Contacts → Google Contacts, one-shot
**Actual:**
- Pulled: <N> from Notion
- Created: <M> new Google Contacts
- Updated: <K> existing
- Skipped (no email / internal): <L>
- Failed: <F> (with per-contact reason in this section)
**Status:** ✅ Complete | ⚠️ Partial | 🔴 Failed
```

Then: `git add memory/ && git commit -m "contact-migration <ISO>" && git push origin main`.

## Hard rules
- NEVER run twice. `state.json.contact_migration_completed_at` gates this.
- NEVER include @hpe.com or @anthropicidentity.com contacts. Internal accounts, not migration targets.
- ON duplicate-email errors: Find + Update, never create duplicates.
- NEVER overwrite a populated Google Contact field with empty data. Only fill in blanks.
- ALWAYS write the runlog and commit before exit.
