# Skill: digest

**Last updated:** 2026-05-07

Composes the Slack digest message Liam reads to keep on top of his deals. Called by Phase 5 of the `assistant` routine. Read `skills/slack.md` first for channel + reply mechanics.

## Format

Slack Block Kit message posted to `#desk-monkey` (or DM if channel unavailable). Two parts:

### Header

```
🦧 Desk Monkey digest — Wed May 7, 2:00pm MT
3 items need you, 2 things on autopilot, 1 meeting in the next 4 hours.
```

### Body — top 5 items, numbered

Each item is one Block Kit `section` block:

```
[#1] DEFCON 2 — Anthropic Identity (Stage POC)
Last touched 8 days ago. Chris went silent after the Apr 29 setup call. Harry meeting hasn't recurred.

→ Proposed: nudge Chris (draft staged).
> "Hey Chris, want to make sure I'm not getting in your way on the Notion build..."

Reply: send 1 | rewrite 1 <instructions> | skip 1
```

Format rules per item:
- DEFCON in the header (1-5 from `gotchas.md`)
- 1-2 line context: what's outstanding + why it surfaced
- Bullet arrow `→ Proposed:` followed by one-line action
- For Gmail items: blockquote draft preview, first 2-3 lines max
- For Calendar items: blockquote subject + start time + attendees
- For Stage-gate items: blockquote evidence
- Reply hints inline so Liam doesn't have to remember the DSL

### Footer

```
+2 more in thread (DEFCON 4-5)
Reply patterns: `send 1, 3` | `skip 2` | `rewrite 4 tighter, ask about budget` | `send invite 5`
Last digest: 11:00am MT (3 hours ago, 2 replies executed)
```

If the run had nothing to surface:
```
🦧 Desk Monkey digest — <time>
✅ Nothing to surface this run. Inbox clean, no drift, next meeting at <time>.
Last digest: <time> (<delta>, <replies executed>)
```

## Composition rules

- **Max 5 items in the main message.** Items 6+ go in a thread reply with abbreviated format (header + one-line context, no draft preview).
- **DEFCON 1-2 always surface.** No suppression.
- **DEFCON 3 surfaces if no Liam-action in past 24h.** If Liam has touched the deal in the last day, hold for next run.
- **DEFCON 4-5 only surface in the 17:00 and 20:00 runs** (end-of-day windows).
- **Already-skipped items** stay suppressed for 24h after the `skip N` reply. Re-surface after that if still drifting.
- **Already-deferred items** wait until the new deadline before re-surfacing.
- **Voice rules from `humanizer.md` apply to draft previews.** No em dashes. No AI vocab. Polite + concise.

## Item type templates

### Left-on-read prospect (Gmail draft)

```
[#N] DEFCON 3 — <Deal name> (Stage <stage>)
<Person> hasn't replied since <date> (<X>d ago) on the <topic> thread.

→ Proposed: nudge (draft staged in Gmail Drafts).
> <first 2 lines of draft body>

Reply: send N | rewrite N <instructions> | skip N
```

### Counterpart commitment overdue (nudge draft + Liam Attio task)

```
[#N] DEFCON 2 — <Deal name>: <commitment> overdue
<Counterpart> agreed by <deadline> to <action>. <Days> overdue. No <verification path> evidence.

→ Proposed: nudge (draft staged) + tracking task in Attio.
> <first 2 lines of nudge>

Reply: send N | rewrite N <instructions> | skip N | defer N <date>
```

### Calendar invite proposal

```
[#N] DEFCON 2 — Schedule with <person>
<Reason for meeting>. Liam's available: <window 1>, <window 2>, <window 3>.

→ Proposed: send invite for <best window>.
Subject: <subject>
Attendees: <list>
Meeting link: Google Meet (auto-generated on send)

Reply: send invite N <window> | rewrite N <change> | skip N
```

### Stage gate met

```
[#N] DEFCON 2 — <Deal name>: ready for <new stage>
<Evidence>: <e.g. "Champion Tests Passed: 3/4. Channel change ✓, buying-process ✓, completed action items ✓.">

→ Proposed: advance Stage from <current> to <new>. Final call is yours.

Reply: advance N | hold N | skip N
```

Note: Stage advancement is an explicit Liam decision per CLAUDE.md hard NEVER. The digest only flags readiness; the routine doesn't auto-advance even on a `send` reply.

### Meeting prep

```
[#N] DEFCON 3 — <Meeting title> at <time>
<Attendees>. Last touch on <Deal>: <date>. Open items going in: <list>.

→ Proposed: read the brief Note before the call.
Brief: <link to Attio Note BRIEF-<event_id>>

Reply: skip N (no action; brief stays on the Deal)
```

## Slack tool contract

- **Channel:** `#desk-monkey` private channel in Liam's Slack workspace. Falls back to DM if channel doesn't exist.
- **Tool:** Slack MCP `chat.postMessage` (capability name; runtime confirms exact tool surface during preflight).
- **Reply read:** `conversations.history` with `oldest=<last digest timestamp>`.
- **Format:** Block Kit JSON. Use `header`, `section`, `divider` blocks. No buttons (replies are free-form text).
- **Threading:** Each digest is a top-level message. Liam's replies in the digest's thread are read by the next run. The next digest is a NEW top-level message, not a thread reply, so the channel stays scannable.
- **Confirmation reply:** After Phase 6 executes Liam's instructions, post a thread reply on the original digest: "Executed: send 1, 2 / skip 3 / revising 4. Full log in runlog."

## Slack message volume + delivery

5 digests per weekday + thread confirmations = ~10 messages/day in `#desk-monkey`. If that's noisy, Liam can mute the channel and check on his cadence; digests are passive surfacing, not push-interrupt.

## Failure modes

- **Slack MCP unavailable:** fall back to email digest to liam@deskmonkeyai.com with subject `Desk Monkey digest — <time>`. Same content, plain markdown body. Log `BLOCKED_TOOL_GAP: Slack chat.postMessage` in runlog.
- **`#desk-monkey` channel doesn't exist:** post via DM. Log `Slack channel #desk-monkey not found, used DM fallback`.
- **Digest empty:** still post the `✅ Nothing to surface` message so Liam knows the routine ran. Silent failure is worse than noise here.