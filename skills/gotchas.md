# Gotchas

> STUB. Liam: paste your canonical `gotchas.md` from `~/Documents/Claude/Projects/Desk Monkey Brain Context/` over this file when ready. The list below is the recovered rule set, kept here as the starting point.

## Hard NEVERs

- NEVER auto-send anything externally. Drafts only. Email = Gmail Drafts. iMessage = Notion Send Now flag + send-worker.
- NEVER advance Deal Stage without explicit verbal commitment from the buyer in a transcript.
- NEVER touch Forecast Category. Liam's call.
- NEVER overwrite Deal Evolution. Always append, newest at top.
- NEVER dump raw transcripts into Notion. Summaries only (2-3 lines).
- NEVER tag HPE-internal threads with a Deal. Channel=Note, Direction=Internal, Status=Logged, no Deal relation.
- NEVER process the same Fireflies transcript twice. Skill State `last_fireflies_id` gates this.
- NEVER create duplicate DEFCON Tasks. Dedupe by Deal + Task title before creating.
- NEVER overwrite a populated Google Contact field with empty data on contact-migration. Only fill blanks.

## Status lifecycle (Activity Log)

- **Open Action** → entry awaiting Liam's review or send.
- **Send Now** → marked for the local send-worker to deliver. ONLY for Channel=Text + Direction=Out. Email drafts go in Gmail Drafts (no Send Now needed).
- **Logged** → completed. Calls and Meetings always land here. Internal HPE notes always land here with Direction=Internal and no Deal relation.
- **Needs Routing** → routine couldn't resolve to a Deal/Contact. Surface for Liam to manually attach.
- **📦 Archive view** → final resting state.

## Status lifecycle (DEFCON Tasks)

- **Open** → not yet started.
- **In Progress** → actively being worked.
- **Blocked** → waiting on someone or something. Notes must say what.
- **Done** → completed.
- **Killed** → decided not to do. Different from Done.

## DEFCON levels

- **DEFCON 1** → drop everything. Same-day. Compelling event triggered.
- **DEFCON 2** → today or tomorrow. Active deal in late stage.
- **DEFCON 3** → this week. Standard follow-up cadence.
- **DEFCON 4** → next week or two. Maintenance, slow-moving deal.
- **DEFCON 5** → nice to have. Backlog.

## Dedupe rules

- Activity Log: don't duplicate for the same Fireflies transcript or same Gmail thread (use thread_id and meeting_id).
- DEFCON Tasks: dedupe by Deal relation + Task title before creating (use search/query).
- Skill State: upsert (create if missing, update if exists) — never create duplicate Key+Skill rows.
- Audit: never re-audit a week already in audit.md or covered by Skill State `last_audit_week`.

## Scope (what's in vs out)

- **coworker** = Fireflies + Gmail + Calendar sweep, Activity Log + DEFCON Tasks + Deal updates. NOT iMessage. NOT Slack. NOT outbound Apollo sequences (yet).
- **contact-migration** = one-shot Notion → Google Contacts. NOT a recurring sync.
- **self-audit** = bounded read of last 200 lines of runlog + stale-deal sweep. Never reads the whole runlog.
- **triage** (local) = iMessage only. Drafts based on Activity Log Instruction field.
- **send-worker** (local) = iMessage send only via Notion Send Now queue.

## Common drift to watch for

- Drafting in customer-service voice (em dashes, AI vocab, "circling back"). Read aloud test.
- Updating Stage on weak signals like "send me an email about it".
- Creating duplicate Activity Log entries for the same Fireflies transcript or Gmail thread.
- Forgetting to update Skill State after a successful run (next run reprocesses everything).
- Letting `[IN_PROGRESS]` runlog stubs stay [IN_PROGRESS] because the routine errored mid-run. Wrap work in try/recover, write 🔴 Failed report on exception.
- DEFCON Tasks created with Owner=blank but no Notes explaining what counterpart owes (verification sweep can't act).

## HPE-internal filter

These are HPE day-job activity, NOT Desk Monkey deal activity:
- Selling With training references
- HPE channel updates
- Strata / Optiv references
- Any meeting where every external attendee is on `@hpe.com`

Log as Channel=Note, Direction=Internal, Status=Logged, no Deal relation, or skip entirely if low-signal.
