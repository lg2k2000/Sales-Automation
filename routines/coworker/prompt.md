# desk-monkey-coworker

Tactical loop. Runs 2x weekdays. Looks at the past 24-36 hours. Three sweeps: Fireflies, Gmail unanswered, Calendar lookahead. Writes to Attio (Notes for activity, Tasks for action items, Deal Record updates for high-confidence evidence). Drops drafts into Gmail Drafts. Never sends.

Daily-rollup work (counterpart verification, Deal property rollups across multiple activities, prospecting, draft pruning) lives in `daily-review`, not here. This routine stays narrow and fast.

## Step 0 — Stub runlog

Append to `memory/runlog.md`:
```
## <ISO> — desk-monkey-coworker [IN_PROGRESS]
```

## Step 1 — Fireflies sweep

`mcp__Fireflies__fireflies_get_transcripts` (sorted desc by date, limit 20). Filter to transcripts whose meeting date is in the last 36 hours.

For each transcript: follow `skills/parse-call.md` end-to-end. That skill handles non-Desk-Monkey filter, dedupe via Attio Note title-prefix query, Deal/Person resolution, Attio Note creation, Tasks creation, Deal Record updates, draft email.

Track per-transcript outcome: processed / skipped (non-Desk-Monkey) / skipped (already logged) / unresolved / failed.

## Step 2 — Gmail unanswered sweep

### Step 2a — Auto-classify and archive system noise

Before processing for Deal threads, sweep for system noise patterns. This replaces the missing Gmail filter creation API (Zapier doesn't expose it; routine logic does it instead).

`mcp__Gmail__search_threads` query: `in:inbox newer_than:1d`. For each thread, classify by sender:

- **Newsletters**: subject or body contains `unsubscribe`, OR sender domain matches known newsletter list (substack, mailchimp, sendgrid, etc.) → Zapier `Add Label to Email` with label `System/Newsletters` + Zapier `Archive Email`.
- **Receipts**: sender matches billing pattern (stripe, billing@, invoice@, receipt@, no-reply+billing, anthropic billing, notion billing, apollo billing, attio billing, google payments) → label `System/Receipts` + archive.
- **Notifications**: `from:notifications@github.com`, `from:calendar-notification@google.com`, app alerts (no-reply, donotreply patterns) → label `System/Notifications` + archive.

Skip if thread is already labeled with one of the System/* labels. No double-labeling.

### Step 2b — Unanswered Deal threads

`mcp__Gmail__search_threads` query: `in:inbox newer_than:7d -from:me`.

For each thread:
1. `mcp__Gmail__get_thread` and find the last message sender.
2. Skip if last sender is Liam (waiting on them) or doesn't match a Person Record in Attio.
3. Match sender email to a Person Record (`mcp__Attio__find_record` people, by email_address). Get the linked active Deal Record. Skip if no Deal match (general inbox noise is not your job).
4. **Dedupe via Attio Notes**: `mcp__Attio__list_notes` for the Deal Record. If any Note title starts with `EMAIL-<gmail_thread_id>`, skip (already logged + drafted).
5. If thread last activity was >24h ago AND Deal Stage NOT IN [Closed Won, Closed Lost, On Hold]:
   - Create Attio Note attached to Deal + Person:
     - Title: `EMAIL-<gmail_thread_id> — <thread subject>`
     - Body: `[Channel: Email] [Direction: In] [Status: Open Action] [Source: <gmail_permalink>] [Last reply: <date> by <sender>]` plus Summary (1-2 line gist of the last message) plus Action Items.
   - Create draft response with `mcp__Gmail__create_draft` (set `thread_id` so it threads). Apply humanizer voice. Address what they asked, propose the next step.
   - Apply relationship-type label via Zapier `Add Label to Email` based on Deal Stage:
     - Closed Won → `Customer`
     - Active stage (Discovery/Qualified/POC/Co-Building/Proposal/Negotiation/Procurement) or side state (Closed Lost / Nurture / Unresponsive) → `Prospect`
     - Person's Relationship is Referrer or Ally → `Partner`
   - Plus state label `_Action` (mutually exclusive with `_Waiting`).

Note: counterpart-silence-over-5-days nudge logic lives in `daily-review`, not here.

## Step 3 — Calendar lookahead

`mcp__Google-Calendar__list_events` for next 24 hours.

For each event:
1. Skip if all attendees internal (no external attendee email matching a Person Record).
2. Match external attendees to Person Records → Deal. If matched:
   - **Dedupe via Attio Notes**: `mcp__Attio__list_notes` for the Deal. If any Note title starts with `BRIEF-<calendar_event_id>` AND is from today, skip (brief already written today).
   - Create Attio Note attached to Deal: title `BRIEF-<calendar_event_id> — <meeting title>`. Body includes last touchpoint date+channel, Open Questions (from Deal attribute), Identified Pain (from Deal attribute), Next Action (from Deal attribute), last 3 Notes attached to this Deal.
3. **Create Attio Task** with content `[DEFCON <2|3>] [Meeting Prep] [Liam] prep for <meeting title>`:
   - DEFCON 2 if same-day, 3 if tomorrow
   - deadline_at = meeting start time
   - assignees = [liam@deskmonkeyai.com]
   - linked_records = [Deal Record, Person Record]
   - Skip if a Task with this title prefix already exists Open for this Deal.

## Step 4 — Final runlog + commit

Replace `[IN_PROGRESS]` with:

```
## <ISO> — desk-monkey-coworker
**Expected:** sweep last 24h Fireflies / Gmail / Calendar, log + draft + Attio Tasks, no sends
**Actual:**
- Fireflies: <P> processed via parse-call, <S> skipped (non-Desk-Monkey/already-logged), <U> unresolved, <F> failed
- Gmail noise: <N> threads classified + archived (newsletters/receipts/notifications)
- Gmail Deal threads: <N> scanned, <M> flagged in Attio, <K> drafts created
- Calendar: <N> external meetings, <M> briefs, <K> prep tasks
**Status:** ✅ Healthy | ⚠️ Drift | 🔴 Failed
**Drift:** [...]
**Errors:** [...]
```

Then: `git add memory/runlog.md && git commit -m "coworker run <ISO>" && git push origin main`.

## Hard NEVERs

- NEVER auto-send. Drafts only.
- NEVER skip the Attio Note dedupe query. Idempotency is the whole reason it works to run 2x/day.
- NEVER advance Deal Stage from coworker. That's daily-review's call (and only on hard evidence).
- NEVER touch Forecast Category.
- NEVER process the same transcript or thread twice.
- NEVER write to Notion CRM DBs (Contacts/Deals/Activity Log/DEFCON Tasks). Those are legacy. Attio is canonical now. Notion only for Projects + knowledge.
- ALWAYS write the runlog before exit.
- ALWAYS commit + push `memory/runlog.md` at end.
