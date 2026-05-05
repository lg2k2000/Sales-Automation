# Gotchas

> STUB. Liam: paste your canonical `gotchas.md` from `~/Documents/Claude/Projects/Desk Monkey Brain Context/` over this file. The list below is the recovered rule set from CLAUDE.md, kept here as the starting point.

## Status lifecycle (Notion Activity Log)

- **Open Action** → entry awaiting Liam's review or send.
- **Send Now** → marked for the local send-worker to deliver. ONLY for Channel=Text + Direction=Out. Email drafts go in Gmail Drafts (no Send Now needed).
- **Logged** → completed. Calls and Meetings always land here. Internal HPE notes always land here with Direction=Internal and no Deal relation.
- **📦 Archive view** → final resting state.

## Hard NEVERs

- NEVER auto-send anything externally. Drafts only.
- NEVER advance Deal Stage without explicit verbal commitment in a transcript.
- NEVER touch Forecast Category. That's Liam's call.
- NEVER overwrite Deal Evolution. Always append.
- NEVER dump raw transcripts into Notion. Summaries only.
- NEVER process the same Fireflies transcript twice. `state.json.last_fireflies_id` gates this.
- NEVER tag HPE-internal threads with a Deal. Channel=Note, Direction=Internal, Status=Logged, no Deal relation.

## Dedupe

- Activity Log: check Action Items for `[Deal updated <date>]` marker before reprocessing.
- Contact migration: gated by `state.json.contact_migration_completed_at`.
- Audit: never re-audit a week already in `audit.md` — check headers first.

## Scope

- Coworker routine = sweep Fireflies + Gmail + Calendar. Not iMessage. Not Slack. Not Apollo outbound (yet).
- Contact-migration = one-shot. Not a recurring sync.
- Self-audit = bounded read of last 200 lines of runlog. Never reads the whole thing.

## Common drift to watch for

- Drafting in customer-service voice. Read aloud test.
- Updating Stage on weak signals like "send me an email about it".
- Creating duplicate Activity Log entries for the same transcript or thread.
- Forgetting to commit + push memory/ at end of cloud routine. Stop hook will catch it locally; cloud needs to do it explicitly.
- Letting [IN_PROGRESS] runlog stubs stay [IN_PROGRESS] because the routine errored mid-run. Always wrap the work in try/recover and write a 🔴 Failed report on exception.
