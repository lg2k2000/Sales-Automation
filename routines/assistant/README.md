# Routine: assistant

**Last updated:** 2026-05-07
**Status:** active — replaces `coworker` (2x/day) + `daily-review` (1x/day)

## Purpose

The single Desk Monkey routine. Sweeps the world, updates the canonical CRM, scores what needs Liam's attention, drafts the next move, and pushes a digest to Slack. Liam responds in Slack with free-form text; the next run picks up his replies and executes.

## Cadence

5 runs per weekday, scheduled in the Anthropic routine UI (cron strings live there, not in this repo). Documented intent:

- **07:00 MT** morning brief — yesterday's misses, today's calendar, fresh transcripts
- **11:00 MT** late-morning — process AM email, surface follow-ups
- **14:00 MT** afternoon — meeting prep for late-day calls, mid-cycle drift sweep
- **17:00 MT** end of day — tomorrow prep, draft next-day touches, prune AM drafts
- **20:00 MT** wrap — digest of unhandled items, MAP refreshes, runlog seal

Pro plan budget: 5 routine runs/day. Self-audit consumes 1 run on Sundays (still separate).

## Phases (run in fixed order, every invocation)

The routine is one prompt, six phases, executed sequentially. Cross-phase contamination is the main risk — each phase reads only the artifacts from prior phases.

1. **Sweep** — pull deltas across Fireflies / Gmail / Calendar / Apollo / Slack `#desk-monkey` since last run
2. **Update** — apply `parse-call` to new transcripts, label new email, refresh Deal MAP Drafts
3. **Triage** — score each active Deal against drift criteria; build the surface-list
4. **Draft** — pre-build Gmail drafts, Calendar invite proposals, Liam-owned Attio nudge tasks
5. **Digest** — compose and post to Slack `#desk-monkey` per `skills/digest.md`
6. **Execute replies** — read Slack replies since last digest, parse free-form intent, execute on Liam's behalf (per `skills/slack.md`)

See `prompt.md` for the full phase-by-phase prompt body.

## What this replaces

| Old | New |
|---|---|
| `coworker` 2x/day (Fireflies + Gmail + Calendar sweep) | Phases 1-4 of `assistant` |
| `daily-review` 1x/day (prune drafts, verify counterpart commitments, prospect sweep) | Phases 3-4 of `assistant`, plus reply-execution in Phase 6 |
| Drafts pile up in Gmail; Liam has to remember to send | Slack digest with `send N` reply pattern |

`self-audit` (weekly Sunday) and `contact-migration` (one-shot) stay separate.

## Tool contract preflight

At the start of every run, the routine confirms availability of:

- Attio: search-records, list-records, create-record, upsert-record, update-record, list-attribute-definitions, create-note, list-tasks, create-task, update-task, list-lists, add-record-to-list (per `skills/attio-tooling.md`)
- Notion: search, fetch, update-page, query-database-view (for `📈 Deals` MAP Draft updates)
- Fireflies: get_transcripts, get_transcript
- Gmail: search_threads, get_thread, create_draft, list_drafts (send is per-item-authorized via Slack reply only)
- Calendar: list_events, suggest_time, create_event (per-invite-authorized via explicit Liam directive)
- Slack: chat.postMessage, conversations.history (capability names; runtime confirms exact tool surface)
- Zapier (Google Contacts, Gmail send/delete-draft, Apollo where wired) per `skills/attio-tooling.md` fallback rules

Logs `TOOL_CONTRACT` line to `memory/runlog.md` at start of run.

## Hard NEVERs (specific to this routine)

- NEVER advance Deal Stage without explicit verbal commitment (per CLAUDE.md).
- NEVER auto-send Gmail without an explicit Slack reply tagging the item with `send`. Drafts only by default.
- NEVER auto-create Calendar events with attendees without an explicit Slack reply tagging the item with `send invite` (or in-conversation Liam authorization).
- NEVER skip the Slack digest. The digest is the entire point — no surface, no value.
- NEVER process the same Fireflies transcript twice. Attio Note dedupe gates this (per `skills/parse-call.md`).
- NEVER create counterpart-owned Attio Tasks. Counterpart commitments go in the Notion Deal `MAP Draft` field and the meeting Note action items, not Liam's Attio task list.
- ALWAYS write the runlog entry before exit, even on failure (🔴 Failed report on exception).
- ALWAYS commit + push `memory/runlog.md` at end of run.

## Open architectural question

Slack → routine triggering: Anthropic routines run on cron, not webhook. Liam's replies in Slack are picked up on the *next* scheduled run, not in real time. Workarounds being evaluated:

1. **Tighter cadence** — 5x/day is the current answer. Worst-case ~3hr delay between Liam reply and execution.
2. **Zapier sentinel** — Slack message → Zapier → write a "trigger" file in this repo or update a dedicated Notion page. Next routine sees the trigger and prioritizes. Doesn't shorten latency but flags urgency.
3. **Agent SDK migration** — long-term, move to the Anthropic Agent SDK with a Slack event listener for true real-time. Significantly more infra; revisit when latency becomes painful.

For v1: stick with cron-scheduled 5x/day and free-form text replies parsed in Phase 6.