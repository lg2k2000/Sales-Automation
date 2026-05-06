# Skill: parse-call

Convert one Fireflies transcript into Notion records + a draft follow-up email. Called by the `coworker` routine for each new transcript. Apply CLAUDE.md voice rules + `skills/humanizer.md` to all written output.

## Inputs

- A single Fireflies transcript (id, url, title, date, attendees, summary, action items, transcript text)

## Output (in order)

1. Activity Log row in Notion
2. DEFCON Tasks for action items
3. (Optional) Deal page property updates on high-confidence evidence
4. Draft follow-up email in Gmail Drafts

## Step 1 — Non-Desk-Monkey filter

Skip the transcript entirely if NO attendee email matches a row in the Notion Contacts DB AND no attendee is associated with an active Deal.

If the meeting is non-Desk-Monkey but worth keeping a record of (industry briefing, training, internal coordination), log a single Activity Log row: Channel=Note, Direction=Internal, Status=Logged, Activity=`<meeting title>`, Summary=`<2-3 lines>`, no Deal relation. Then return.

## Step 2 — Idempotency check (Activity Log dedupe)

Query Activity Log (`mcp__Notion__notion-query-database-view`) for any row where Summary or Notes contains this transcript's Fireflies URL or transcript_id. If a row exists, return early. We've already processed this one.

## Step 3 — Resolve Deal + Contact

Match each external attendee email to Contacts (`collection://474cee31-b5fe-45e6-906a-b8463eada553`). Contacts have a relation to Deals, so this gives the Deal too.

If no attendee resolves to a Contact: log Activity Log row with Status=Needs Routing, no Deal relation, Summary tagged `[unresolved attendees: <emails>]`. Skip steps 4-6. Surface in runlog as needs-review.

If multiple Deals match (rare): pick the one in the most active Stage (not Closed/Nurture/On Hold). Note the ambiguity in runlog.

## Step 4 — Activity Log row

Create via `mcp__Notion__notion-create-pages` against `collection://b4edca94-b39f-4f9d-9b29-e53cde7b71c6`:

- **Channel**: Meeting
- **Direction**: In
- **Status**: Logged
- **Activity**: meeting title from Fireflies (truncate to 100 chars)
- **Summary**: 2-3 line digest. NEVER raw transcript. Capture: what was discussed, what changed, what's next. Buyer's words for pain. Apply humanizer voice.
- **Action Items**: bulleted, owner per line. Format:
  ```
  - [Liam] draft proposal v2 by Tue 5/13
  - [Craig] send Houston Foam contacts by Fri 5/9
  - [Liam] schedule technical walkthrough w/ James
  ```
- **Raw Content**: leave empty
- **Instruction**: leave empty (that field is for Liam's directives, not the routine)
- **Deal**: relation to resolved Deal
- **Contact**: relation to primary attendee (the Deal's primary contact)
- **Timestamp**: meeting start time
- **Notes** (or appended to Summary): `Fireflies: <url>` so future runs can dedupe.

## Step 5 — DEFCON Tasks for action items

For each action item from Step 4, create a row in `collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e`.

**Liam-owned action item:**
- Task: the action, one line, imperative ("draft proposal v2", "schedule walkthrough w/ James")
- Owner: Liam (person field)
- Status: Open
- DEFCON: default 3. Bump to 2 if "today/ASAP/this week" language. Bump to 1 if same-day commitment to a buyer. Drop to 4-5 only if explicitly "no rush / nice to have".
- Category: Follow-up (default), or Meeting Prep / Client Deliverable / Internal as appropriate
- Source: Meeting
- Due: stated deadline if present in transcript; otherwise +7 days from meeting date
- Deal, Contact: relations set
- Notes: `from <meeting title> on <date> — <one-line context>`

**Counterpart-owned action item:**
- Task: same imperative form ("send Houston Foam contacts")
- Owner: BLANK (this is the marker for daily-review's verification sweep)
- Status: Open
- DEFCON: 3
- Source: Meeting
- Due: stated deadline if present; otherwise +5 days from meeting date
- Deal, Contact: relations set
- Notes: `counterpart-owned: <person> agreed to <action> by <date> on <meeting>. Verify via <how>.`
  - Example: `counterpart-owned: Craig agreed to send Houston Foam contact list by Fri 5/9 on Houston Foam discovery call. Verify via Gmail attachment from cwalicek@ciotech.us.`
  - Example: `counterpart-owned: Andrew agreed to send calendar invite for technical walkthrough by Wed 5/7. Verify via Calendar event with abrink@blueally.com.`

**Dedupe check before creating**: query DEFCON Tasks for Open tasks where Deal=this Deal AND Task title is similar to the new one. If a near-match exists, skip creation, append a note to the existing row instead.

## Step 6 — Deal property updates (high-confidence only)

Only update what the transcript provides hard evidence for. If you'd hedge writing it, skip.

- **Stage**: only on explicit verbal commitment. "Yes let's do a POC starting next month" → advance. "Send me an email about it" → no change.
- **Buyer Behavior Stage**: advance based on observed behavior, not seller activity. Stage 2 (Multithreaded validation) when 2+ stakeholders engaged. Stage 3 (Exec sponsored) when an exec has joined. Stage 4 (Approach agreed) when buyer agrees to your approach before product pitch.
- **Next Action**: replace with the agreed concrete next step.
- **Identified Pain**: set if not yet set and transcript articulates clear pain in buyer's own words.
- **Champion Verdict**: Coach/True Champion only on behavioral evidence (channel change, buying-process disclosure, completed action items, trained value story).
- **Champion Tests Passed**: increment if a new test passed. Append to **Champion Test Evidence**: `<date>: <test name> — <observed behavior>`.
- **Compelling Event**: set if real timing-driven reason surfaces (renewal, fiscal close, board mtg, hire date, regulatory).
- **Cost of Doing Nothing**: set when buyer articulates consequences of inaction.
- **Decision Process**: update on new approval steps, gates, committee members revealed.
- **Open Questions**: remove answered, add new (format: `question — owner — by when`).
- **Future State**: set/refine when buyer describes target outcome.
- **Buyer-Owned Action Ratio**: increment based on action item ownership ratio in this meeting.

**Always append one-line Deal Evolution**: `MMM DD, YYYY: <one-line: what this meeting revealed and what got updated>`. Reverse chronological — newest at top of the field.

## Step 7 — Draft follow-up email

Skip if the transcript is internal-only or unresolved (no Deal).

`mcp__Gmail__create_draft`:
- **To**: external attendees from the meeting (their emails, comma-separated)
- **From**: liam@deskmonkeyai.com
- **Subject**: `Re: <meeting title>` if there was a calendar invite, else `Follow-up — <topic>`
- **Body** (apply humanizer voice. read aloud test):

  ```
  <1-2 line recap of what we agreed>

  Action items:
  - <owner>: <action> by <date>
  - <owner>: <action> by <date>

  <one-line next step ask or close>

  Best
  --
  Liam Glennie
  720-431-2310
  deskmonkeyai.com
  ```

  The signature block is mandatory. First-name-only sign-offs are not acceptable on email drafts. See `skills/humanizer.md` for the full signature spec.

This is the **first-pass draft**. After creating it, append `[first-pass draft pending prune]` to the Activity Log Meeting row's Action Items field. The `daily-review` routine will revisit later that day and rewrite the draft tighter (cut to 30-50% of length, focus on the single most important next-step ask). The wait between first-pass (within hours of meeting) and pruned-pass (evening) builds in human pacing — a draft sent 30 seconds after a meeting reads as AI; a tightened draft sent the next morning reads as a person.

  Examples:

  > Good draft: `Recap of today: you're sold on the approach, blocker is getting Janet's approval before procurement. Action items: me — send the one-pager Janet can forward by Tuesday. You — pull Janet into the next call. Next thing: scheduling that call once you have her on board.`

  > Bad draft (don't write this): `Hi Andrew, just wanted to circle back on our conversation today. I really enjoyed our discussion about your team's robust workflow. Per our discussion, I will send the one-pager so we can leverage Janet's approval. Looking forward to hearing back from you. Best, Liam`

## Hard NEVERs (specific to this skill)

- NEVER process a transcript twice. Activity Log query in Step 2 gates it.
- NEVER write Liam's name as "Liam Glennie" when his own draft. Just sign "Liam" or no signature (drafts already have his Gmail signature).
- NEVER advance Stage on weak signals.
- NEVER overwrite Deal Evolution. Append.
- NEVER skip the DEFCON Tasks step. Action items hitting Notion is the entire point.
- NEVER mark a counterpart-owned task with Owner=Liam. Owner=blank is the verification sweep's signal.
