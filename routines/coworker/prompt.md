# desk-monkey-coworker

Tactical loop. Runs 3x weekdays. Looks at the past 24 hours. Three sweeps: Fireflies, Gmail unanswered, Calendar lookahead. Logs to Activity Log, creates DEFCON Tasks for action items, drops drafts into Gmail Drafts. Never sends.

Daily-rollup work (counterpart verification, Deal property updates from accumulated activity, prospecting) lives in `daily-review`, not here. This routine stays narrow and fast.

## Step 0 — Stub runlog

Append to `memory/runlog.md`:
```
## <ISO> — desk-monkey-coworker [IN_PROGRESS]
```

## Step 1 — Fireflies sweep

`mcp__Fireflies__fireflies_get_transcripts` (sorted desc by date, limit 20). Filter to transcripts whose meeting date is in the last 36 hours (covers gaps between runs).

For each transcript: follow `skills/parse-call.md` end-to-end. That skill handles non-Desk-Monkey filter, dedupe via Activity Log query, Deal/Contact resolution, Activity Log row creation, DEFCON Tasks, Deal updates, draft email.

Track per-transcript outcome: processed / skipped (non-Desk-Monkey) / skipped (already logged) / unresolved (Needs Routing) / failed.

## Step 2 — Gmail unanswered sweep

### Step 2a — Auto-classify and archive system noise

Before processing for Deal threads, sweep for system noise patterns. This replaces the missing Gmail filter creation API (Zapier doesn't expose it; routine logic does it instead, with a worst-case 4h delay between this run and the next).

`mcp__Gmail__search_threads` query: `in:inbox newer_than:1d`. For each thread, classify by sender:

- **Newsletters**: subject or body contains `unsubscribe`, OR sender domain matches known newsletter list (substack, mailchimp, sendgrid, etc.) → Zapier `Add Label to Email` with label `System/Newsletters` + Zapier `Archive Email`.
- **Receipts**: sender matches billing pattern (stripe, billing@, invoice@, receipt@, no-reply+billing, anthropic billing, notion billing, apollo billing, google payments) → label `System/Receipts` + archive.
- **Notifications**: `from:notifications@github.com`, `from:calendar-notification@google.com`, app alerts (no-reply, donotreply patterns) → label `System/Notifications` + archive.

Skip if thread is already labeled with one of the System/* labels. No double-labeling.

### Step 2b — Unanswered Deal threads

`mcp__Gmail__search_threads` query: `in:inbox newer_than:7d -from:me`.

For each thread:
1. `mcp__Gmail__get_thread` and find the last message sender.
2. Skip if last sender is Liam (waiting on them) or doesn't match a row in Notion Contacts.
3. Match sender email to Contacts → Deals. Skip if no Deal match (general inbox noise is not your job).
4. **Dedupe via Activity Log**: query for an Email row with the Gmail thread ID in Notes/Summary AND Status=Open Action. If exists, skip (already logged + drafted).
5. If thread last activity was >24h ago AND Deal Stage NOT IN [Closed Won, Closed Lost, On Hold]:
   - Create Activity Log row: Channel=Email, Direction=In, Status=Open Action, Activity=thread subject, Summary=last message gist (1-2 lines), Deal+Contact set, Notes=`Gmail thread: <permalink-or-id>`.
   - Create draft response with `mcp__Gmail__create_draft` (set `thread_id` so it threads). Apply humanizer voice. Address what they asked, propose the next step.
   - Apply Deal-aligned label via Zapier `Add Label to Email` (direct Gmail label MCPs are disconnected). Use the relationship-type scheme from `skills/inbox.md`: `Customer` if Deal Stage = Closed Won; `Prospect` if Deal Stage is any active stage (Discovery / Qualified / POC / Co-Building / Proposal / Negotiation / Procurement) or side states (Closed Lost / Nurture / Unresponsive); `Partner` if the Contact's Relationship is Referrer or Ally; `Vendor` if the Contact is a paid service. Plus `_Action` (needs Liam's reply) or `_Waiting` (Liam already replied), mutually exclusive.

Note: counterpart-silence-over-5-days nudge logic lives in `daily-review`, not here.

## Step 3 — Calendar lookahead

`mcp__Google-Calendar__list_events` for next 24 hours.

For each event:
1. Skip if all attendees internal.
2. Match external attendees to Contacts → Deal. If matched:
   - **Dedupe via Activity Log**: query for a Note row with this Calendar event ID in Notes AND Channel=Note AND Status=Logged AND today's Timestamp. If exists, skip (brief already written today).
   - Write a brief on the Deal page (`mcp__Notion__notion-update-page`): last touchpoint date+channel, Open Questions, Identified Pain, Next Action, last 3 Activity Log entries for this Deal.
   - Log Activity Log row: Channel=Note, Direction=Internal, Status=Logged, Activity=`Brief written for <meeting title>`, Notes=`Calendar event: <id>`, Deal set.
3. **Create DEFCON Task** `prep for <meeting title>` if no Open Task already exists with this exact title for this Deal:
   - Owner=Liam, Status=Open, DEFCON=2 if same-day, 3 if tomorrow, Category=Meeting Prep, Source=Internal, Due=meeting date, Deal+Contact set.

## Step 4 — Final runlog + commit

Replace `[IN_PROGRESS]` with:

```
## <ISO> — desk-monkey-coworker
**Expected:** sweep last 24h Fireflies / Gmail / Calendar, log + draft + DEFCON Tasks, no sends
**Actual:**
- Fireflies: <P> processed via parse-call, <S> skipped (non-Desk-Monkey/already-logged), <U> unresolved, <F> failed
- Gmail: <N> threads scanned, <M> flagged, <K> drafts created
- Calendar: <N> external meetings, <M> briefs, <K> prep tasks
**Status:** ✅ Healthy | ⚠️ Drift | 🔴 Failed
**Drift:** [...]
**Errors:** [...]
```

Then: `git add memory/runlog.md && git commit -m "coworker run <ISO>" && git push origin main`.

## Hard NEVERs

- NEVER auto-send. Drafts only.
- NEVER skip the Activity Log dedupe query. Idempotency is the whole reason it works to run 3x/day.
- NEVER advance Deal Stage from coworker. That's daily-review's call (and only on hard evidence).
- NEVER touch Forecast Category.
- NEVER process the same transcript or thread twice.
- ALWAYS write the runlog before exit.
- ALWAYS commit + push `memory/runlog.md` at end.
