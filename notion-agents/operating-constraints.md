# Operating constraints

What every agent must respect. Bake into every agent's instructions block.

## Hard NEVERs (no agent ever does these autonomously)

- **NEVER auto-send Gmail externally.** Drafts only. Send only on explicit Liam comment `send <N>` for that specific item.
- **NEVER auto-create Calendar events with external attendees.** Send invite only on explicit Liam comment `send invite <N>`. Per-invite authorization, not blanket.
- **NEVER advance Deal Stage.** Even on `advance N` — write a readiness flag in Deal Evolution; Liam advances manually.
- **NEVER modify Forecast Category.** Liam's call exclusively.
- **NEVER overwrite Deal Evolution.** Always append. Newest at top.
- **NEVER dump raw transcripts into Activity Log Summary.** 2-3 line digests in Liam's voice only.
- **NEVER process the same source twice.** Title-prefix dedupe (see Idempotency below) is the gate.
- **NEVER tag a non-Desk-Monkey thread with a Deal.** If sender / attendee doesn't match a Contact in an active Deal, skip.
- **NEVER create duplicate Tasks.** Dedupe by Deal + title prefix.
- **NEVER create counterpart-owned Tasks.** Counterpart commitments live in the meeting Note Action Items + Deal MAP Draft. Liam's task list stays his own.
- **NEVER skip the Daily Brief.** Even when nothing surfaces, post "✅ Nothing to surface. Inbox clean. Next meeting at <time>."
- **NEVER delete receipts or billing email.**
- **NEVER delete email from a sender on an active (non-Closed) Deal.**
- **NEVER skip Liam's voice rules.** Read `voice-rules.md` and `banned-list.md` before any external draft.

## Idempotency rules

Every write must be safe to re-run. Use unique source-ID title prefixes:

| Source | Title prefix |
|---|---|
| Notion meeting note | `MTG-<note_id_short>` |
| Gmail thread | `EMAIL-<thread_id>` |
| Calendar event | `BRIEF-<event_id>` |
| Manual call log | `CALL-<YYYY-MM-DD>` |
| Manual note | `NOTE-<YYYY-MM-DD>` |

Before creating any row, search the parent's children for a row whose title starts with the source-ID prefix. If found: skip the create.

## Rate limits

- **Email Sweeper:** at most 100 threads per run.
- **Inbox Sanitation:** at most 50 deletions per run. Surface remainder in next Daily Brief.
- **Meeting Linker:** at most 20 meeting notes per run.
- **All agents:** if Notion API rate limits hit, back off exponentially (2s, 4s, 8s, 16s) up to 4 retries. Then log + exit clean.
- **Reply Executor:** at most 10 emails sent per run.

## Allowlists (never delete senders)

**Receipts (NEVER delete, NEVER unsubscribe):**
stripe.com, anthropic.com, notion.so, attio.com, apollo.io, slack.com, zapier.com, google.com (billing), microsoft.com (billing), aws.amazon.com (billing), linkedin.com (billing only)

**Active deal contacts (NEVER delete):**
Any sender on a Contact linked to a non-Closed Deal. Re-checked on every Inbox Sanitation run.

**Personal (NEVER touch):**
Empty by default. Populate via `allowlist <sender>` Liam comment on Daily Brief.

## Status lifecycles

**Activity Log Status:**
- `Logged` — done, archived
- `Open Action` — needs Liam reply or action
- `Needs Routing` — agent low-confidence; Liam disambiguates
- `Send Now` — Reply Executor send-worker picks up within 15 min and sends. ONLY for Channel=Email or Text AND Direction=Out

**Deal Stage:**
New / Discovery / Qualified / POC / Co-Building / Proposal / Negotiation / Procurement / Closed Won / Closed Lost / Nurture / On Hold / Unresponsive. Agents never advance Stage. Flag readiness via a `[DEFCON 2] [Internal] [Liam] <Deal>: ready for <new stage> — <evidence>` task.

## Voice gate

Every external draft (email body, Daily Brief items, page content) passes this checklist before saving:

1. Read aloud test
2. No banned vocab (see `banned-list.md`)
3. No em dashes
4. No customer-service openers
5. No commanding-the-recipient phrasing
6. Email signature block on every email draft
7. Specific time windows offered for any scheduling proposal
8. Anti-AI pass: name 3-4 tells, fix them, re-read

If any check fails, redo.

## Stop conditions

The agent halts and writes a Run Log row with Status=Drift if:

1. A required Notion DB / page is missing or trashed
2. A Connection (Gmail / Calendar) has expired tokens
3. A required attribute is renamed / dropped
4. Rate limit retries exhausted
5. An unrecognized field option appears in a select / status (`FIELD_OPTION_GAP`)
6. The agent's prompt produces low-confidence intent on a destructive action

Never silently fail. Every run produces a Run Log row.
