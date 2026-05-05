# desk-monkey-coworker

You run 3x weekdays. Sweep Fireflies, Gmail, Calendar. Log to Notion (Activity Log + DEFCON Tasks + Deal updates). Draft follow-ups in Gmail Drafts. Never auto-send. Read CLAUDE.md before starting.

## Step 0 — Read state from Notion + stub runlog

Query Notion Skill State DB (`collection://7e0c32b3-112c-4bb0-b058-ee588e8ca921`) for rows where Skill=coworker. Read keys: `last_fireflies_id`, `last_gmail_sweep`, `last_calendar_sweep`. If a row doesn't exist, treat the value as null.

Append to `memory/runlog.md`:
```
## <ISO> — desk-monkey-coworker [IN_PROGRESS]
```

## Step 1 — Fireflies sweep

`mcp__Fireflies__fireflies_get_transcripts` (sorted desc by date, limit 20). For each transcript with id newer than `last_fireflies_id`:

1. Fetch attendee list. Apply HPE-internal filter: skip if every external attendee is on `@hpe.com` or there are no external attendees.
2. Resolve to Deal: match attendee email → Contacts → Deal relation. If no match, log to Activity Log without Deal relation and flag in runlog as needs-review.
3. `mcp__Fireflies__fireflies_get_summary` for the digest + action items.
4. Create Activity Log row (`mcp__Notion__notion-create-pages` against `collection://b4edca94-b39f-4f9d-9b29-e53cde7b71c6`):
   - Channel=Meeting, Direction=In, Status=Logged
   - Activity = meeting title from Fireflies
   - Summary = 2-3 line digest (NOT raw transcript)
   - Action Items = bulleted list with owner per line
   - Deal, Contact relations set
   - Timestamp = meeting start time
5. **Action items → DEFCON Tasks** (`collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e`):
   - For each action item assigned to Liam: create row with Task=action, Owner=Liam, Status=Open, DEFCON=2 (default; 1 if "ASAP/today" language, 3 if "next week"), Category=Follow-up (or Meeting Prep / Client Deliverable per content), Source=Meeting, Due=stated deadline if present, Deal+Contact relations set, Notes=link to Activity Log row.
   - For each action item assigned to a counterpart: create row with Task=action, Owner=blank, Status=Open, DEFCON=3, Source=Meeting, Due=stated deadline, Notes="counterpart-owned: <person> agreed to <action> by <date>. Verify via <how>" — this is what enables the verification sweep in Step 5.
   - Dedupe: check existing Open DEFCON Tasks with same Deal relation + similar Task title before creating.
6. **Deal updates** (only on high-confidence evidence — see "Deal property updates" below). Always append a one-line Deal Evolution entry: `MMM DD, YYYY: <what got revealed and changed>`.
7. **Draft follow-up email** (`mcp__Gmail__create_draft`):
   - Voice rules from `skills/humanizer.md` apply. No em dashes. No AI vocab. Read aloud test.
   - Structure: 1-2 line recap → action items (owner per line) → next step ask.
   - To: external attendees from the meeting. From: liam@deskmonkeyai.com.
   - Subject: `Re: <meeting title>` or `Follow-up — <topic>`.
8. Update Skill State: write `last_fireflies_id` = this transcript's id (Skill=coworker).

## Step 2 — Gmail unanswered sweep

`mcp__Gmail__search_threads` query: `in:inbox newer_than:7d -from:me`. For each thread:

1. `mcp__Gmail__get_thread`. Identify last sender.
2. Skip if last sender is Liam (waiting on them) or internal (`@hpe.com` / `@anthropicidentity.com`).
3. Match sender to Contacts → Deal. Skip if no Deal match.
4. If thread last activity >24h ago AND not on hold:
   - Create Activity Log row: Channel=Email, Direction=In, Status=Open Action, Deal+Contact relations, Summary=last message gist, Activity=thread subject.
   - Create draft response via `mcp__Gmail__create_draft` with `thread_id` set so it threads correctly.
   - Apply Deal-aligned Gmail label via Zapier `Add Label to Email` (direct Gmail label MCP is unavailable).
5. If thread last activity from counterpart is >5 days old AND Deal Stage NOT IN [Closed Won, Closed Lost, Nurture, On Hold, Unresponsive]:
   - Create DEFCON Task: Task=`soft nudge <contact> on <subject>`, Owner=Liam, DEFCON=3, Category=Follow-up, Source=Email, Notes=`last reply <date>, Gmail thread <permalink>`, Deal+Contact set.

Update Skill State: `last_gmail_sweep` = current ISO timestamp (Skill=coworker).

## Step 3 — Calendar lookahead

`mcp__Google-Calendar__list_events` for next 24 hours.

For each event:
1. Skip if all attendees internal.
2. Match external attendees to Deal. If matched:
   - Write a brief on the Deal page (`mcp__Notion__notion-update-page`): last touchpoint date+channel, Open Questions, Identified Pain, Next Action, last 3 Activity Log entries.
   - Skip brief if Deal already has a brief from today (idempotency check via Deal Evolution last line).
3. Create DEFCON Task `prep for <meeting title>` if not already present:
   - Owner=Liam, Status=Open, DEFCON=2 if same-day, 3 if tomorrow, Category=Meeting Prep, Source=Internal, Due=meeting start date, Deal+Contact set.

Update Skill State: `last_calendar_sweep` = current ISO timestamp.

## Step 4 — Counterpart commitment verification

Query DEFCON Tasks where Owner is empty AND Due < today AND Status=Open.

For each:
1. Read Notes for verification path (e.g. "Craig will send invite by Friday — verify via Calendar").
2. Search the appropriate surface (Gmail for emails sent, Calendar for invites received, Drive for shared docs).
3. If proof found: flip Task Status=Done. Append note: `<ISO> verified via <where>`.
4. If no proof: create a new DEFCON Task: Task=`nudge <person> on <original task>`, Owner=Liam, DEFCON=2 (active Deal) or 3 (Nurture), Category=Follow-up, Source=Internal, Due=today, Notes=`originally due <date>, no proof of completion`. Mark the original Task Status=Blocked.

## Step 5 — Final runlog + commit

Replace `[IN_PROGRESS]` with:

```
## <ISO> — desk-monkey-coworker
**Expected:** sweep Fireflies / Gmail / Calendar, log + draft + DEFCON Tasks, no sends
**Actual:**
- Fireflies: <N> processed, <M> Activity Log rows, <K> Deal updates, <L> drafts, <T> DEFCON Tasks created
- Gmail: <N> threads scanned, <M> flagged, <K> drafts, <L> nudge tasks
- Calendar: <N> external meetings, <M> briefs, <K> prep tasks
- Verification: <N> tasks checked, <M> verified, <K> nudge tasks created
**Status:** ✅ Healthy | ⚠️ Drift | 🔴 Failed
**Drift:** [...]
**Errors:** [...]
```

Then commit: `git add memory/runlog.md && git commit -m "coworker run <ISO>" && git push origin main`.

## Deal property updates (high-confidence only)

Apply to the linked Deal page on Step 1.6:

- **Stage**: only on explicit verbal commitment ("yes let's do a POC starting next month"). "Send me an email about it" = no change.
- **Buyer Behavior Stage**: advance based on observed behavior, not seller activity. Stage 2 (Multithreaded validation) when 2+ stakeholders engaged. Stage 3 (Exec sponsored) when an exec has joined a call. Stage 4 (Approach agreed) when buyer agrees to your approach before product pitch.
- **Next Action**: replace with the agreed concrete next step.
- **Identified Pain**: set if not yet set and transcript articulates clear pain in buyer's words.
- **Champion Verdict**: Coach/True Champion only with behavioral evidence (channel change, buying-process disclosure, completed action items, trained value story).
- **Champion Tests Passed**: increment if a new test passed in this transcript. Append to **Champion Test Evidence**: `<date>: <test name> — <observed behavior>`.
- **Compelling Event**: set if real timing-driven reason surfaces (renewal, fiscal close, board mtg, hire date, regulatory).
- **Cost of Doing Nothing**: set when buyer articulates consequences of inaction.
- **Decision Process**: update on new approval steps, gates, committee members revealed.
- **Open Questions**: remove answered, add new (format: `question — owner — by when`).
- **Future State**: set or refine when buyer describes target outcome.
- **Buyer-Owned Action Ratio**: increment based on action item ownership in the meeting.
- **Last Touch**: auto-updates on edit; don't set explicitly.

Always append one-line **Deal Evolution**: `MMM DD, YYYY: <one-line: what got revealed and changed>`. Reverse chronological — newest at top.

## Hard NEVERs
- NEVER auto-send any email or message externally
- NEVER advance Stage without explicit commitment
- NEVER invent facts not in the transcript or email
- NEVER process the same Fireflies transcript twice (Skill State `last_fireflies_id` gates this)
- NEVER touch Forecast Category — Liam's call
- NEVER overwrite Deal Evolution. Always append, newest at top.
- NEVER dump raw transcripts into Notion. Summaries only.
- NEVER create duplicate DEFCON Tasks. Dedupe by Deal + Task title before creating.
- ALWAYS write the runlog before exit
- ALWAYS commit + push `memory/runlog.md` at end (state stays in Notion Skill State, not in repo)
