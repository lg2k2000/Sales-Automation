# Skill: parse-call

Convert one Fireflies transcript into Attio + Gmail records. Called by the `coworker` routine for each new transcript. Apply CLAUDE.md voice rules + `skills/humanizer.md` to all written output.

**Before Step 1**, read `skills/attio-tooling.md` and run Attio preflight if the calling routine has not already done so. All Attio actions below use the capability names from that skill (search records, list records, create record, upsert record, update record, list-attribute-definitions, create-note, list-tasks, create-task, update-task) — not hardcoded MCP function names. If a required Attio capability is missing in the runtime, log `BLOCKED_TOOL_GAP` for that capability and skip only the blocked write step.

## Inputs

- A single Fireflies transcript (id, url, title, date, attendees, summary, action items, transcript text)

## Output (in order)

1. Attio Note attached to the matching Deal Record (and Person Record)
2. Attio Tasks for action items (linked to Deal Record)
3. (Optional) Attio Deal Record attribute updates on high-confidence evidence
4. Draft follow-up email in Gmail Drafts

## Step 1 — Non-Desk-Monkey filter

Skip the transcript entirely if NO attendee email matches a Person Record in Attio AND no attendee is associated with a Deal Record (via the Person → Deal linked-records relationship).

If the meeting is non-Desk-Monkey but worth keeping a record of (industry briefing, training, internal coordination), attach a Note to a designated "Misc / Internal" Person Record with title `NOTE-<YYYY-MM-DD> — <meeting title>`. Then return.

## Step 2 — Idempotency check (Attio Note dedupe)

Use the Attio notes/list notes capability after runtime preflight, filtered to the matching Deal Record. Check for any existing Note whose title starts with `MTG-<fireflies_id>`. If found, return early — already processed.

## Step 3 — Resolve Deal + Person

Match each external attendee email to a Person Record (use the Attio search/list records capability with object_id=`people`, filter by email_address). The Person Record has linked Deal Records — use the most active Deal (Stage NOT IN [Closed Won, Closed Lost, Nurture, On Hold, Unresponsive]).

If no Person Record matches an attendee: do NOT auto-create a Person here. Surface to runlog as `unresolved-attendee` and either route the Note to the "Unrouted" placeholder Record or skip per CLAUDE.md non-Desk-Monkey filter.

If a Person matches but no active Deal exists, follow the Deal creation protocol in `skills/attio-tooling.md`:
- If the transcript shows strong buying signal (real sales discussion, pricing/proposal, POC, migration, renewal pain, explicit next step), proceed with Deal creation per that protocol — search Companies, search Deals, list-attribute-definitions for `deals`, then create-record / upsert-record. Build the attribute map using only fields that exist and select options that are valid in the workspace.
- If confidence is below 0.80, do NOT create a Deal. Create a Liam-owned Task instead: `[DEFCON 3] [Pipeline Build] [Liam] Review whether to create Deal for <Company>`, linked to the Person Record. Attach the Note to the Person Record.

If no attendee resolves to a Person Record at all: attach the Note to a designated "Unrouted" placeholder Record, set the Note title to `MTG-<fireflies_id> — UNROUTED — <meeting title>`, and surface in runlog as needs-review. Skip steps 4-6.

If multiple Deals match (rare): pick the most active. Note the ambiguity in runlog.

## Step 4 — Attio Note creation

Use the Attio create-note capability with parent_object=`deals`, parent_record_id=<Deal record ID>, format=`markdown`:

**Title:** `MTG-<fireflies_id> — <meeting title>` (truncate meeting title to fit ~80 chars)

**Content:**
```
[Channel: Meeting]
[Direction: In]
[Status: Logged]
[Source: <fireflies_url>]
[Date: <YYYY-MM-DD HH:MM>]
[Attendees: <comma-separated emails>]

## Summary
<2-3 line digest. Capture: what was discussed, what changed, what's next. Buyer's words for pain. Apply humanizer voice. NEVER raw transcript.>

## Action Items
- [Liam] <action> by <date>
- [<counterpart name>] <action> by <date> — verify via <how>

## Raw notes
<optional: any quote or detail worth keeping that doesn't fit Summary>
```

Also attach the same Note (or a brief reference) to the primary attendee's Person Record so it shows up on the contact's timeline.

## Step 5 — Attio Tasks for action items

For each action item from Step 4, use the Attio create-task capability. Encode DEFCON + Category + Owner in the content/title prefix since Attio Tasks don't support custom attributes natively.

**Liam-owned action item:**
```
content: "[DEFCON <2 or 3>] [<Category>] [Liam] <action>

From <meeting title> on <date>.
<one-line context>

Linked Note: MTG-<fireflies_id>"

deadline_at: stated deadline if present in transcript; otherwise +7 days from meeting date
assignees: ["liam@deskmonkeyai.com"]
linked_records: [Deal Record, Person Record]
```

DEFCON default is 3. Bump to 2 if "today/ASAP/this week" language. Bump to 1 if same-day commitment to a buyer. Drop to 4-5 only if explicitly "no rush / nice to have".

Categories: Apollo Setup / Pipeline Build / Email Infrastructure / Client Deliverable / Data Cleanup / Meeting Prep / Follow-up / Internal.

**Counterpart-owned action item:**
```
content: "[DEFCON 3] [<Category>] [<counterpart name>] <action>

counterpart-owned: <person> agreed to <action> by <date> on <meeting>.
Verify via <Gmail attachment from <email> | Calendar event with <email> | Notion shared doc | etc.>

Linked Note: MTG-<fireflies_id>"

deadline_at: stated deadline if present; otherwise +5 days from meeting date
assignees: []   # empty assignees is the marker for daily-review's verification sweep
linked_records: [Deal Record, Person Record]
```

**Dedupe before creating:** use the Attio list-tasks capability for this Deal. If an Open Task with similar title (same Owner + similar action) exists, skip creation; instead append a note to that Task's content via the Attio update-task capability.

## Step 6 — Deal Record attribute updates (high-confidence only)

Use the Attio update-record capability on the Deal Record, after calling `list-attribute-definitions` for object `deals` to confirm attribute slugs and allowed select options. Only update what the transcript provides hard evidence for. If you'd hedge writing it, skip. If a target select value isn't in the allowed options for the workspace, log `FIELD_OPTION_GAP` and skip that field.

- **Stage**: only on explicit verbal commitment. "Yes let's do a POC starting next month" → advance. "Send me an email about it" → no change.
- **Buyer Behavior Stage**: advance based on observed behavior, not seller activity.
- **Next Action**: replace with the agreed concrete next step.
- **Identified Pain**: set if not yet set and transcript articulates clear pain in buyer's own words.
- **Champion Verdict**: Coach/True Champion only on behavioral evidence.
- **Champion Tests Passed**: increment if a new test passed. Append to **Champion Test Evidence**: `<date>: <test name> — <observed behavior>`.
- **Compelling Event**: set if real timing-driven reason surfaces.
- **Cost of Doing Nothing**: set when buyer articulates consequences of inaction.
- **Decision Process**: update on new approval steps, gates, committee members revealed.
- **Open Questions**: remove answered, add new (format: `question — owner — by when`).
- **Future State**: set/refine when buyer describes target outcome.
- **Buyer-Owned Action Ratio**: increment based on action item ownership ratio in this meeting.
- **Last Touch**: auto-updates on edit; don't set explicitly.

**Always append one-line Deal Evolution**: `MMM DD, YYYY: <one-line: what this meeting revealed and what got updated>`. Reverse chronological — newest at top of the field. Use the Deal Evolution custom attribute (text type, append, never overwrite).

## Step 7 — Draft follow-up email

Skip if the transcript is internal-only or unresolved (no Deal).

`mcp__Gmail__create_draft`:
- **To**: external attendees from the meeting (their emails)
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

This is the **first-pass draft**. After creating it, append a marker line to the Attio Note's content: `[first-pass draft pending prune]`. The `daily-review` routine will revisit later that day and rewrite the draft tighter (cut to 30-50% of length, focus on the single most important next-step ask). The wait between first-pass (within hours of meeting) and pruned-pass (evening) builds in human pacing — a draft sent 30 seconds after a meeting reads as AI; a tightened draft sent the next morning reads as a person.

## Hard NEVERs (specific to this skill)

- NEVER process a transcript twice. Attio Note dedupe in Step 2 gates it.
- NEVER write Liam's name as "Liam Glennie" when his own draft. Just sign with the standard signature block (which has Liam Glennie as part of the structured block).
- NEVER advance Stage on weak signals.
- NEVER overwrite Deal Evolution. Append.
- NEVER skip the Tasks step. Action items hitting Attio is the entire point.
- NEVER mark a counterpart-owned task with assignees=[Liam]. Empty assignees is the verification sweep's signal.
- NEVER dump the raw transcript into the Attio Note. Summary only.
