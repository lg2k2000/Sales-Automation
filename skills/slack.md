# Skill: slack

**Last updated:** 2026-05-07

Slack is the primary surfacing channel for the `assistant` routine. Liam reads digests in `#desk-monkey`, replies with free-form text, and the next routine run parses + executes those replies.

## Channel setup

- **Primary:** `#desk-monkey` (private channel in Liam's Slack workspace)
- **Fallback:** DM to Liam if channel doesn't exist or routine can't post to it
- **Read scope:** the routine reads only its own digest threads + their replies; doesn't read the rest of the workspace

## Reply DSL — what Liam can type

Free-form natural English. The routine parses for these patterns (case-insensitive). Liam doesn't need to use exact syntax — close-enough phrasing works.

### Send actions

```
send 1
send 1, 2, 3
send 2 and 4
yes to 1, send it
```
For each Gmail draft item: send via Zapier Gmail Send action.
For each Calendar item: create_event with notificationLevel=ALL.

### Skip actions

```
skip 3
no on 2
ignore 4 for now
hold 5
```
Suppress the item for 24h. Logged in runlog. Re-surfaces in next digest if still drifting after the suppression window.

### Rewrite actions

```
rewrite 1 tighter
rewrite 2 to ask about budget
redo 3 more casual
4 is too long, cut the second paragraph
```
Update the linked Gmail draft per the instruction. Surface the revised draft in the next digest as `[#N revised]`. Don't send until Liam reviews and replies `send N`.

### Schedule / defer actions

```
defer 1 to next Monday
push 2 to May 15
not until Friday on 3
```
Push the linked Attio Task deadline forward. Item won't re-surface until the new date.

### Calendar invite actions

```
send invite 4
yes to invite 4 at 2pm
send invite 5 with the 10am window
```
For Calendar items: create the event with notificationLevel=ALL, attendees from the proposed list, Google Meet auto-generated. Send to all proposed attendees.

### Escalate / urgency actions

```
escalate 2
bump 3 to defcon 1
this is urgent on 1
```
Bump DEFCON level on the linked Task or item.

### Multi-action in one message

```
send 1 and 2, skip 3, rewrite 4 to drop the second paragraph, send invite 5 at 10am
```
Parse all actions in order. Execute each. Confirmation thread reply summarizes.

### Free text not matching DSL

If Liam types something that doesn't match a known pattern (e.g., "what about the Authonomy thing? did Chris ever reply?"), don't try to interpret. Log as `unparsed reply` in runlog and surface in the next digest as:

```
[#N from prior digest] Liam asked: "<their text>"
→ Proposed: <best-effort answer or clarifying question>
Reply: <next-step DSL>
```

## Confirmation reply pattern

After Phase 6 executes Liam's instructions, post a thread reply on the original digest:

```
✅ Executed:
- Sent draft #1 to mileskurtz@hfpusa.com
- Sent invite #4 (May 9 2pm MT, Google Meet attached)
- Revised draft #2 — preview in next digest
- Skipped #3 (suppressed 24h)
Full run report in runlog.
```

This gives Liam an immediate audit trail without having to check the runlog.

## Message + reply volume

Per weekday:
- 5 top-level digests (one per scheduled run)
- ~5 confirmation replies (one per run that executed Liam actions)
- Liam's own replies (variable)

If volume gets noisy, Liam can mute `#desk-monkey` and check on his own cadence. Digests are passive surfacing.

## Privacy + safety

- The routine never posts customer-facing content (drafts to be sent, transcript summaries, etc.) anywhere outside `#desk-monkey` or Attio. Slack is a private workspace.
- Each digest is self-contained — the routine doesn't post raw transcript bodies, full email threads, or sensitive Selling With Deal attributes (e.g., Personal Stakes, Show-Stoppers) in the digest. Reference them by link to Attio.
- Reply parsing is restricted to messages from Liam's authenticated user. Other workspace members' messages are ignored.

## Tool contract

Capability names (runtime confirms exact tool surface during preflight):
- `chat.postMessage` — post digest + confirmation replies
- `conversations.history` — read Liam's replies since last digest timestamp
- `users.identify` — confirm reply sender is Liam (not another workspace member)
- `chat.update` (optional) — edit a prior digest if a draft gets revised mid-cycle

Log `TOOL_CONTRACT Slack: postMessage=<yes/no> history=<yes/no>` in runlog at preflight.

## When Slack is unavailable

If `chat.postMessage` is missing or fails:
1. Log `BLOCKED_TOOL_GAP: Slack chat.postMessage` in runlog
2. Fall back to email digest: `mcp__Gmail__create_draft` to liam@deskmonkeyai.com, subject `Desk Monkey digest — <ISO time>`, body = same content as Slack digest but plain markdown
3. Liam reads + replies in his own Gmail; the next routine reads liam's sent messages to that thread
4. Surface the gap in the digest itself: "Slack offline today — replying to this email instead."

## Open: real-time triggering

Anthropic routines run on cron, not webhook. So Liam's replies sit in `#desk-monkey` until the next scheduled run picks them up (worst case ~3hr at 5x/day).

If real-time becomes important:
- Zapier sentinel: Slack message → Zapier scenario → write a `memory/triggers/<timestamp>.md` file in this repo. The next routine sees the file at preflight and bumps urgency.
- Agent SDK migration: long-term, move Phase 6 (reply execution) into the Anthropic Agent SDK with a Slack event listener for true real-time. Phases 1-5 stay on cron.

For v1: live with the ~3hr latency. Document open question in routine README.