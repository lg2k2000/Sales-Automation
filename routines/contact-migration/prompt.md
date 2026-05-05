# desk-monkey-contact-migration

One-shot routine. Pulls Notion Contacts and creates Google Contacts via Zapier. Idempotent via Skill State DB.

## Step 0 — Idempotency check + stub runlog

Query Notion Skill State DB (`collection://7e0c32b3-112c-4bb0-b058-ee588e8ca921`) for Skill=contact-migration, Key=`completed_at`. If a row exists with a non-null Value, exit immediately and write to runlog: "Migration already completed at <Value>. Manual reset required to re-run (delete that row in Skill State)."

Otherwise append:
```
## <ISO> — desk-monkey-contact-migration [IN_PROGRESS]
```

## Step 1 — Pull Notion Contacts

`mcp__Notion__notion-fetch` on `collection://474cee31-b5fe-45e6-906a-b8463eada553` to get the schema. Then query the data source for all rows.

For each contact, extract: Full name, Email, Phone, Company, Title, Linked Deal (relation), Notes (truncate to 200 chars).

Skip:
- Empty email
- `@hpe.com` or `@anthropicidentity.com` (internal — not migration targets)

## Step 2 — Ensure Gmail labels exist

`mcp__Gmail__list_labels`. For any of these missing, `mcp__Gmail__create_label`:
- `Anthropic-Identity`, `Authonomy`, `Houston-Foam`, `Blue-Ally`, `MyVP`
- `_Inbox/Action`, `_Inbox/Waiting`

Add a label per active Deal as Liam's deal list grows. Pull active deal names from `collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4` (Stage NOT IN [Closed Won, Closed Lost, Nurture]).

## Step 3 — Create Google Contacts via Zapier

For each Notion contact, call `mcp__Zapier__execute_zapier_write_action` with the `Google Contacts: Create Contact` action. Pass: name, email, phone, company, title, notes.

On duplicate-email error: call `Google Contacts: Find Contact` by email, then `Google Contacts: Update Contact` to fill any blank fields without overwriting populated ones.

Track per-contact result: created / updated / skipped / failed.

## Step 4 — Mark complete in Skill State

Create or update Skill State row:
- Skill=contact-migration, Key=`completed_at`, Value=ISO timestamp
- Skill=contact-migration, Key=`stats`, Value=JSON `{"pulled":N,"created":M,"updated":K,"skipped":L,"failed":F}`, Notes=optional per-failure reasons

## Step 5 — Final runlog + commit

```
## <ISO> — desk-monkey-contact-migration
**Expected:** all eligible Notion Contacts → Google Contacts, one-shot
**Actual:**
- Pulled: <N>
- Created: <M>
- Updated: <K>
- Skipped (no email / internal): <L>
- Failed: <F>
**Status:** ✅ Complete | ⚠️ Partial | 🔴 Failed
```

Commit: `git add memory/runlog.md && git commit -m "contact-migration <ISO>" && git push origin main`.

## Hard rules
- NEVER run twice. Skill State `completed_at` gates this.
- NEVER include internal emails (@hpe.com, @anthropicidentity.com).
- ON duplicate-email errors: Find + Update, never create dupes.
- NEVER overwrite a populated Google Contact field with empty data. Only fill blanks.
- ALWAYS write runlog and commit before exit.
