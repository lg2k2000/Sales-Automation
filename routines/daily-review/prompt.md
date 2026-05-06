# desk-monkey-daily-review

Rollup. Runs once per weekday evening. Four jobs: prune today's first-pass meeting drafts (anti-AI-pacing), counterpart commitment verification, Deal Record rollups from today's Notes, left-on-read prospecting.

This routine is the strategic counterpart to `coworker`. coworker logs and drafts in real-time during the day. daily-review steps back, looks at the day's accumulated Attio Notes, and decides what changed at the Deal level.

## Step 0 — Stub runlog

Append to `memory/runlog.md`:
```
## <ISO> — desk-monkey-daily-review [IN_PROGRESS]
```

## Step 1 — Counterpart commitment verification

Query Attio Tasks where:
- assignees is empty (counterpart-owned marker)
- deadline_at < today
- is_completed = false

For each:
1. Read the Task content for the verification path (e.g. `Verify via Gmail attachment from cwalicek@ciotech.us` or `Verify via Calendar event with abrink@blueally.com`).
2. Run the appropriate check:
   - **Gmail attachment**: `mcp__Gmail__search_threads` with query `from:<email> has:attachment newer_than:<X>d`. Open the most recent matching thread. Confirm the attachment matches what was promised.
   - **Calendar invite**: `mcp__Google-Calendar__list_events` filtered to events with the named attendee in the next 14 days.
   - **Email response**: search Gmail for any reply from that contact since the meeting date.
   - **Other**: best-effort search per the description; if unverifiable, treat as no-proof.
3. **Proof found**: `mcp__Attio__update_task` with is_completed=true. Append to Task content: `<ISO> verified via <where> (<short evidence>)`. Done.
4. **No proof**: create a follow-up Task for Liam:
   - content: `[DEFCON 2 if Deal Stage active, else 3] [Follow-up] [Liam] nudge <person> on <original task>`
   - deadline_at: today
   - assignees: [liam@deskmonkeyai.com]
   - linked_records: same as original Task (Deal + Person)
   - description: `originally due <date>, no proof of completion. Original Task: <link>.`
   - Mark the original Task content with appended note: `<ISO> verification failed; Liam-owned nudge created (link: <new task URL>)`. Update is_completed=false but content shows status=Blocked semantically.

## Step 2 — Prune today's first-pass meeting drafts (anti-AI-pacing)

Query Attio Notes where:
- title starts with `MTG-` (meeting Note)
- created today
- content contains `[first-pass draft pending prune]`

For each match:

1. Find the corresponding Gmail draft `coworker` created. Look it up via `mcp__Gmail__list_drafts` filtered to today + matching subject (`Re: <meeting title>`) + matching recipient.
2. Read the existing draft body. Apply the prune target: rewrite to 30-50% of the original length. The pruned version keeps:
   - **One** line recap of what got agreed (not three)
   - **One** action item — the single most important one. Their action if it's the gating step. Liam's action if Liam owes the next move.
   - **One** sentence next-step ask or close
   - The signature block (always)
3. Drop:
   - Full bulleted action item lists (collapse to the one that matters)
   - Pleasantries beyond the bare minimum
   - Restating context the recipient already remembers
   - Anything that sounds like an AI summary
4. Replace the Gmail draft: delete the old draft via Zapier `Delete Email` (use the message_id from `list_drafts`), create the new draft via `mcp__Gmail__create_draft` with the same `to`, `subject`, and `replyToMessageId` if applicable.
5. Update the Attio Note: `mcp__Attio__update_record_attributes` (or use a Note-update endpoint if available) — replace `[first-pass draft pending prune]` in the content with `[draft pruned <ISO>]`.

Apply `skills/humanizer.md` voice rules. Read the pruned version aloud. If it doesn't sound like a peer texting a peer (with a signature), prune more.

**Why this exists:** A draft sent 30 seconds after a meeting reads as AI. A tightened draft the recipient sees the next morning reads as a person who took time to think. The wait between first-pass (within hours of meeting) and pruned-pass (this evening) builds in that human pacing.

## Step 3 — Deal Record updates rolled up from today

Query Attio Notes where:
- created today (00:00 local onwards)
- linked to a Deal Record
- title starts with one of: `MTG-`, `EMAIL-`, `CALL-` (so we capture meetings, emails, calls; skip system briefs)

Group by Deal Record. For each Deal with new Notes today:

1. Fetch current Deal Record attributes (`mcp__Attio__find_record` by ID, or read directly from the attached Notes' source-of-truth).
2. Read all of today's Notes for this Deal (Summary + Action Items fields).
3. Apply the Deal property update logic from `skills/parse-call.md` Step 6, but rolled up across all of today's activity rather than per-meeting:
   - **Stage**: only on explicit verbal commitment evidenced in any of today's logged activity. If no commitment, no change.
   - **Buyer Behavior Stage**: advance based on observed behavior across the day's activity.
   - **Next Action**: replace with the most recent agreed next step.
   - **Identified Pain**: set if not yet set.
   - **Champion Verdict**: Coach/True Champion only with hard behavioral evidence.
   - **Champion Tests Passed**: increment count + append to **Champion Test Evidence** if a new test passed.
   - **Compelling Event**: set if surfaced.
   - **Cost of Doing Nothing**: set if buyer articulated.
   - **Decision Process**: update if new approval steps revealed.
   - **Open Questions**: remove answered, add new.
   - **Future State**: set/refine if buyer described target.
   - **Buyer-Owned Action Ratio**: recompute based on action items in today's activity (count of empty-assignees Tasks created today / total Tasks created today linked to this Deal).
4. **Always append Deal Evolution**: one line summarizing today, format `MMM DD, YYYY: <one-line: today's net change at the Deal level>`. Newest at top.

If the day's activity surfaced a Stage gate met (3+ Champion Tests Passed → ready for Qualified, or Buyer-Owned Action Ratio ≥ 50% → ready for Co-Building exit), DO NOT auto-advance Stage. Create an Attio Task: content=`[DEFCON 2] [Internal] [Liam] <Deal>: ready for <new stage> — <evidence>`, deadline_at=tomorrow, assignees=[Liam], linked_records=[Deal].

## Step 4 — Left-on-read prospecting sweep

Find Attio Deals where:
- Stage IN [New, Discovery, Qualified, POC] (active early-stage)
- Last sent email from Liam was >7 days ago (search Gmail `from:me to:<contact email> newer_than:14d` and check most recent date)
- No reply from contact since that send

For each, before creating a nudge: query Attio Tasks for an existing Open `[Follow-up] [Liam] soft nudge <contact>` task. If exists, skip. Otherwise create:
- content: `[DEFCON 3] [Follow-up] [Liam] soft nudge <contact name> on <last subject>`
- deadline_at: today + 1
- assignees: [liam@deskmonkeyai.com]
- linked_records: [Deal Record, Person Record]
- description: `last sent <date>, no reply. Gmail thread: <permalink>. Stage=<current Deal stage>.`

## Step 5 — Final runlog + commit

Replace `[IN_PROGRESS]` with:

```
## <ISO> — desk-monkey-daily-review
**Expected:** prune today's first-pass drafts + counterpart verification + Deal updates from today + prospecting sweep
**Actual:**
- Drafts pruned: <P> meeting drafts rewritten tighter
- Verification: <N> tasks checked, <V> verified done, <B> blocked + nudge created
- Deal updates: <D> deals touched, <U> attribute updates, <G> stage gates flagged
- Prospecting: <S> deals scanned, <N> nudge tasks created
**Status:** ✅ Healthy | ⚠️ Drift | 🔴 Failed
**Drift:** [...]
**Errors:** [...]
```

Commit: `git add memory/runlog.md && git commit -m "daily-review <ISO>" && git push origin main`.

## Hard NEVERs

- NEVER advance Deal Stage automatically, even on full-gate-met evidence. Surface it as an Attio Task for Liam.
- NEVER touch Forecast Category.
- NEVER overwrite Deal Evolution. Append, newest at top.
- NEVER mark a counterpart-owned Task complete without proof. Document the proof in Task content.
- NEVER create a duplicate nudge Task. Check existing Open tasks first.
- NEVER write to Notion CRM DBs. Attio is canonical.
- ALWAYS write the runlog before exit.
- ALWAYS commit + push.
