# desk-monkey-daily-review

Rollup. Runs once per weekday evening. Three jobs: counterpart commitment verification, Deal property updates rolled up from today's Activity Log, left-on-read prospecting.

This routine is the strategic counterpart to `coworker`. coworker logs and drafts in real-time during the day. daily-review steps back, looks at the day's accumulated activity, and decides what changed at the Deal level.

## Step 0 — Stub runlog

Append to `memory/runlog.md`:
```
## <ISO> — desk-monkey-daily-review [IN_PROGRESS]
```

## Step 1 — Counterpart commitment verification

Query DEFCON Tasks (`collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e`) for rows where:
- Owner is empty
- Due < today
- Status = Open

For each:
1. Read the Notes field for the verification path (e.g. `Verify via Gmail attachment from cwalicek@ciotech.us` or `Verify via Calendar event with abrink@blueally.com`).
2. Run the appropriate check:
   - **Gmail attachment**: `mcp__Gmail__search_threads` with query `from:<email> has:attachment newer_than:<X>d`. Open the most recent matching thread. Confirm the attachment matches what was promised.
   - **Calendar invite**: `mcp__Google-Calendar__list_events` filtered to events with the named attendee in the next 14 days. Confirm one exists.
   - **Email response**: search Gmail for any reply from that contact since the meeting date.
   - **Other**: best-effort search per the Notes; if unverifiable, treat as no-proof.
3. **Proof found**: update the Task — set Status=Done, append a Notes line `<ISO> verified via <where> (<short evidence>)`. Done.
4. **No proof**: create a follow-up Task for Liam:
   - Task: `nudge <person> on <original task>`
   - Owner: Liam
   - DEFCON: 2 if Deal Stage in [Discovery, Qualified, POC, Co-Building, Proposal, Negotiation, Procurement], else 3
   - Category: Follow-up
   - Source: Internal
   - Due: today (Liam should send the nudge tonight or tomorrow)
   - Deal, Contact: same as original Task
   - Notes: `originally due <date>, no proof of completion. Original DEFCON Task: <link>.`
   - Mark the original Task Status=Blocked, append Notes line `<ISO> verification failed; Liam-owned nudge created`.

## Step 2 — Deal property updates rolled up from today

Query Activity Log for rows where Timestamp >= today (00:00 local) AND Status=Logged AND Channel IN [Meeting, Call, Email] AND a Deal relation exists.

Group by Deal. For each Deal with new activity today:

1. Fetch current Deal page properties (`mcp__Notion__notion-fetch` or query-database-view, lightweight selection).
2. Read all of today's Activity Log rows for this Deal (Summary + Action Items fields).
3. Apply the Deal property update logic from `skills/parse-call.md` Step 6, but rolled up across all of today's activity rather than per-meeting:
   - **Stage**: only on explicit verbal commitment evidenced in any of today's logged activity. If no commitment, no change.
   - **Buyer Behavior Stage**: advance based on observed behavior across the day's activity (multithreaded validation if 2+ stakeholders engaged, exec sponsored if exec joined a call).
   - **Next Action**: replace with the most recent agreed next step.
   - **Identified Pain**: set if not yet set.
   - **Champion Verdict**: Coach/True Champion only with hard behavioral evidence.
   - **Champion Tests Passed**: increment count + append to **Champion Test Evidence** if a new test passed.
   - **Compelling Event**: set if surfaced.
   - **Cost of Doing Nothing**: set if buyer articulated.
   - **Decision Process**: update if new approval steps revealed.
   - **Open Questions**: remove answered, add new.
   - **Future State**: set/refine if buyer described target.
   - **Buyer-Owned Action Ratio**: recompute based on action items in today's activity (count of Owner=blank DEFCON Tasks created today / total action items today).
4. **Always append Deal Evolution**: one line summarizing today, format `MMM DD, YYYY: <one-line: today's net change at the Deal level>`. Newest at top.

If the day's activity surfaced a Stage gate met (3+ Champion Tests Passed → ready for Qualified, or Buyer-Owned Action Ratio ≥ 50% → ready for Co-Building exit), DO NOT auto-advance Stage. Create a DEFCON Task for Liam: Task=`<Deal>: ready for <new stage> — <evidence>`, Owner=Liam, DEFCON=2, Category=Internal, Source=Internal, Notes summarizing the gate evidence.

## Step 3 — Left-on-read prospecting sweep

Find Deals where:
- Stage IN [New, Discovery, Qualified, POC] (active early-stage)
- Last sent email from Liam was >7 days ago (search Gmail `from:me to:<contact email> newer_than:14d` and check most recent date)
- No reply from contact since that send

For each, before creating a nudge: check DEFCON Tasks for an existing Open `soft nudge <contact>` task. If exists, skip. Otherwise create:
- Task: `soft nudge <contact name> on <last subject>`
- Owner: Liam
- DEFCON: 3
- Category: Follow-up
- Source: Email
- Due: today + 1
- Notes: `last sent <date>, no reply. Gmail thread: <permalink>. Stage=<current Deal stage>.`
- Deal, Contact: relations set

## Step 4 — Final runlog + commit

Replace `[IN_PROGRESS]` with:

```
## <ISO> — desk-monkey-daily-review
**Expected:** counterpart verification + Deal updates from today + prospecting sweep
**Actual:**
- Verification: <N> tasks checked, <V> verified done, <B> blocked + nudge created
- Deal updates: <D> deals touched, <U> property updates, <G> stage gates flagged
- Prospecting: <S> deals scanned, <N> nudge tasks created
**Status:** ✅ Healthy | ⚠️ Drift | 🔴 Failed
**Drift:** [...]
**Errors:** [...]
```

Commit: `git add memory/runlog.md && git commit -m "daily-review <ISO>" && git push origin main`.

## Hard NEVERs

- NEVER advance Deal Stage automatically, even on full-gate-met evidence. Surface it as a DEFCON Task for Liam.
- NEVER touch Forecast Category.
- NEVER overwrite Deal Evolution. Append, newest at top.
- NEVER mark a counterpart-owned Task Done without proof. Document the proof in Notes.
- NEVER create a duplicate nudge Task. Check existing Open tasks first.
- ALWAYS write the runlog before exit.
- ALWAYS commit + push.
