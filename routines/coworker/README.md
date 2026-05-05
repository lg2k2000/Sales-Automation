# coworker

The main cloud routine. Runs 3x weekdays (suggested: 7am, 12pm, 6pm). Sweeps Fireflies + Gmail + Calendar, logs activity to Notion, drafts follow-ups, updates Deal pages on high-confidence evidence. Never sends.

## Trigger
- **Schedule:** set in the Anthropic routine UI (suggested `0 7,12,18 * * 1-5`)
- **Repo:** this one
- **Connectors needed:** Notion, Gmail, Google Calendar, Fireflies, Zapier (for Send / Archive / Trash / Label / Mark Read actions)
- **Routine prompt:** `Read routines/coworker/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## What it does

**Fireflies sweep.** New transcripts since `last_fireflies_id`. For each: skip if HPE-internal, resolve to Deal, log to Activity Log, update Deal properties on hard evidence, draft Gmail follow-up.

**Gmail sweep.** Inbox threads from the past 7 days where the last message is from a Deal contact and Liam hasn't replied. Drafts a response in Gmail Drafts, labels the thread per Deal, logs to Activity Log if unanswered >24h.

**Calendar lookahead.** Next 24h of external meetings. Writes a brief on the matching Deal page so Liam has context before walking into the meeting.

## State

Tracks last-processed cursors in `memory/state.json`:
- `last_fireflies_id`
- `last_gmail_sweep`
- `last_calendar_sweep`

This prevents reprocessing across runs.

## Hard NEVERs
- Never auto-send externally
- Never advance Deal Stage without explicit commitment
- Never reprocess a Fireflies transcript
- Never touch Forecast Category

## See also
- `prompt.md` — exact instructions
- `../../CLAUDE.md` — working memory
- `../../memory/runlog.md` — run history
- `../../memory/state.json` — cross-run cursors
