# Fireflies sync

Pulls new Fireflies transcripts since the last run, matches each to a Notion Deal or Project, logs the meeting, and updates the parent record.

## State

State page ID: {{AUTOMATION_STATE_PAGE_ID}}

- Read last_fireflies_sync_at before fetching transcripts.
- Update last_fireflies_sync_at to the current timestamp after the run completes, regardless of whether every transcript processed successfully. Failed transcripts go to the triage view, not back into the queue.

## Notion database IDs

- Deals: {{DEALS_DB_ID}}
- Projects: {{PROJECTS_DB_ID}}
- Activity Log: {{ACTIVITY_LOG_DB_ID}}
- Contacts: {{CONTACTS_DB_ID}}

## Pulling transcripts

1. Fetch the state page and read last_fireflies_sync_at.
2. Query Fireflies for transcripts updated since that timestamp.
3. Cap the run at 20 transcripts. If more exist, process the oldest 20 and log how many were skipped.

## Per-transcript flow

1. Pull the full transcript via Fireflies MCP using the meeting ID.
2. Check Activity Log for an existing entry tied to this Fireflies meeting ID. If found, skip the transcript.
3. Match to a parent record:
   - Search Deals by attendee email exact match.
   - If no hit, search Deals by attendee email domain.
   - If still no hit, search Deals by company name pulled from the transcript or attendee org.
   - If no Deal matches, run the same three searches against Projects.
   - If nothing matches, set entry status to needs triage and skip the parent update.
4. Create one Activity Log entry with:
   - Meeting date
   - Attendees
   - Fireflies meeting URL
   - 3-5 line summary
   - Action items as bulleted text
   - Link to parent Deal or Project
   - Status: synced (or needs triage)
5. Update the parent record:
   - Bump Last Touch to the meeting date.
   - Replace Next Action only if the transcript contains a clearer commitment than what is already there.
   - Append a dated one-line summary to Deal Evolution.
6. For any attendee not already in Contacts, create a Contact record and link it to the parent.

## Hard rules

- Never overwrite Deal Stage. Liam owns stage changes.
- Never duplicate Activity Log entries. Always check by Fireflies meeting ID first.
- Never invent action items. If the transcript does not contain a clear commitment, leave the action items field empty.
- If matching is ambiguous (two Deals could match), set status to needs triage rather than guessing.
- Do not draft emails. Email drafting is out of scope for this skill in v1.

## Run report

After processing, post a comment on the state page with:
- Transcripts processed
- Matched to Deals
- Matched to Projects
- Sent to triage
- Errors hit

## Out of scope for v1

- Email drafting
- Calendar event matching
- DocuSign milestone logging

These get added in later versions once the matching logic is trustworthy.
