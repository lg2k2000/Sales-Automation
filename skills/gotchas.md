# Gotchas

> STUB. Liam: paste your canonical `gotchas.md` from `~/Documents/Claude/Projects/Desk Monkey Brain Context/` over this file when ready. The list below is the recovered rule set, kept here as the starting point.

## Hard NEVERs

- NEVER auto-send anything externally. Drafts only. Email = Gmail Drafts. Liam clicks Send.
- NEVER advance Deal Stage without explicit verbal commitment from the buyer in a transcript.
- NEVER touch Forecast Category. Liam's call.
- NEVER overwrite Deal Evolution. Always append, newest at top.
- NEVER dump raw transcripts into Notion. Summaries only (2-3 lines).
- NEVER tag a non-Desk-Monkey thread with a Deal. If no attendee or sender matches a Notion Contact or Deal, log Channel=Note Direction=Internal Status=Logged with no Deal relation, or skip entirely.
- NEVER process the same Fireflies transcript twice. Skill State `last_fireflies_id` gates this.
- NEVER create duplicate DEFCON Tasks. Dedupe by Deal + Task title before creating.
- NEVER overwrite a populated Google Contact field with empty data on contact-migration. Only fill blanks.

## Status lifecycle (Activity Log)

- **Open Action** → entry awaiting Liam's review or send.
- **Send Now** → reserved Notion status value (no current routine writes or reads it). Was the iMessage send-worker queue marker; that flow is not currently wired.
- **Logged** → completed. Calls and Meetings always land here. Non-Desk-Monkey internal notes always land here with Direction=Internal and no Deal relation.
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

- **coworker** = Fireflies + Gmail + Calendar sweep, Activity Log + DEFCON Tasks + Deal updates + first-pass meeting drafts + inbox noise classification. NOT iMessage. NOT Slack. NOT outbound Apollo sequences (yet).
- **daily-review** = evening rollup. Prunes today's first-pass meeting drafts. Verifies counterpart commitments. Rolls Deal property updates across today's activity. Left-on-read prospecting.
- **self-audit** = bounded read of last 200 lines of runlog + stale-deal sweep. Never reads the whole runlog.
- **contact-migration** = one-shot Notion → Google Contacts. NOT a recurring sync.

## Common drift to watch for

- Drafting in customer-service voice (em dashes, AI vocab, "circling back"). Read aloud test.
- Updating Stage on weak signals like "send me an email about it".
- Creating duplicate Activity Log entries for the same Fireflies transcript or Gmail thread.
- Forgetting to update Skill State after a successful run (next run reprocesses everything).
- Letting `[IN_PROGRESS]` runlog stubs stay [IN_PROGRESS] because the routine errored mid-run. Wrap work in try/recover, write 🔴 Failed report on exception.
- DEFCON Tasks created with Owner=blank but no Notes explaining what counterpart owes (verification sweep can't act).

## Non-Desk-Monkey filter

If a thread, transcript, or calendar event doesn't involve a Notion Contact or Deal, it's not Desk Monkey work. Don't try to route it.

A meeting/email/transcript counts as Desk Monkey work if at least one external attendee or sender matches:
- A row in the Notion Contacts DB (`collection://474cee31-b5fe-45e6-906a-b8463eada553`), OR
- A relation on an active Deal in the Notion Deals DB (`collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4`)

If neither matches: skip entirely if low-signal, or log as Channel=Note, Direction=Internal, Status=Logged, no Deal relation if there's something worth keeping.

Examples that get skipped:
- Email noise from any non-Contact domain
- Internal coordination calls with people not in Contacts
- Generic notifications, newsletters, system mail
