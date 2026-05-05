# desk-monkey-contact-migration

One-shot routine. Pulls Notion Contacts and creates Google Contacts via Zapier. Idempotent — checks Activity Log for a prior completion event before running.

**PRECONDITION**: Google Contacts must be enabled in the Zapier MCP config (Create / Find / Update Contact actions). Currently NOT enabled. Routine will fail at Step 3 if these aren't available.

## Step 0 — Idempotency check + stub runlog

Query Activity Log (`collection://b4edca94-b39f-4f9d-9b29-e53cde7b71c6`) for any row where:
- Channel = Note
- Direction = Internal
- Activity contains `contact-migration completed`

If a row exists, exit immediately and write to runlog:
```
## <ISO> — desk-monkey-contact-migration [SKIPPED]
**Reason:** Activity Log contains completion event from <prior date>. Manual reset required to re-run (delete that Activity Log row).
```

Otherwise append to runlog:
```
## <ISO> — desk-monkey-contact-migration [IN_PROGRESS]
```

## Step 1 — Pull Notion Contacts

`mcp__Notion__notion-fetch` on `collection://474cee31-b5fe-45e6-906a-b8463eada553` to confirm schema. Then query the data source for all rows.

For each contact, extract: Full name, Email (primary), Phone, Company, Title, Linked Deal (relation), Notes (truncate to 200 chars).

Skip:
- Empty email (can't create a Google Contact without it)
- `@hpe.com` or `@anthropicidentity.com` (internal — not migration targets)

## Step 2 — Ensure Gmail labels exist

`mcp__Gmail__list_labels`. For any of these missing, `mcp__Gmail__create_label`:
- One per active Deal: `Anthropic-Identity`, `Authonomy`, `Houston-Foam`, `Blue-Ally`, `MyVP` (pull live from Deals where Stage NOT IN [Closed Won, Closed Lost, Nurture])
- Triage state: `_Inbox/Action`, `_Inbox/Waiting`

## Step 3 — Create Google Contacts via Zapier

For each Notion contact: `mcp__Zapier__execute_zapier_write_action` with `Google Contacts: Create Contact`. Pass: name, email, phone, company, title, notes.

On duplicate-email error: call `Google Contacts: Find Contact` by email, then `Google Contacts: Update Contact` to fill any blank fields without overwriting populated ones.

Track per-contact result: created / updated / skipped / failed (with reason).

If `Google Contacts: Create Contact` is not available in Zapier (i.e. wasn't enabled in the Zapier MCP config): abort the routine, write a 🔴 Failed runlog entry explaining the precondition, exit. Do NOT mark migration completed.

## Step 4 — Mark complete in Activity Log

Create one Activity Log row:
- Channel = Note
- Direction = Internal
- Status = Logged
- Activity = `contact-migration completed`
- Summary = `Pulled <N>, created <M>, updated <K>, skipped <L>, failed <F>`
- Action Items = empty
- Timestamp = now

This row is the idempotency marker for Step 0.

## Step 5 — Final runlog + commit

```
## <ISO> — desk-monkey-contact-migration
**Expected:** all eligible Notion Contacts → Google Contacts, one-shot
**Actual:**
- Pulled: <N>
- Created: <M>
- Updated: <K>
- Skipped (no email / internal): <L>
- Failed: <F> (reasons inline)
**Status:** ✅ Complete | ⚠️ Partial | 🔴 Failed
```

Commit: `git add memory/runlog.md && git commit -m "contact-migration <ISO>" && git push origin main`.

## Hard rules

- NEVER run twice. Step 0 Activity Log check gates this.
- NEVER include @hpe.com or @anthropicidentity.com contacts.
- ON duplicate-email errors: Find + Update, never create dupes.
- NEVER overwrite a populated Google Contact field with empty data. Only fill blanks.
- If Zapier Google Contacts actions aren't enabled, abort and surface clearly. Don't write the completion marker.
