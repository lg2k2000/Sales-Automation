# Desk Monkey Routines

Sales-automation orchestration for Liam Glennie. Cloud routines (Anthropic Claude Code on the web) handle Fireflies + Gmail + Calendar + Notion sweeps. Notion is canonical state and dedupe ledger. The repo is the operational scratchpad.

See `architecture.md` for the data flow diagram and `CLAUDE.md` for working memory loaded into every session.

## Layout

```
.
├── CLAUDE.md             auto-loaded into every session
├── architecture.md       data flow + tool layer
├── decisions.md          design-decisions log
├── skills/               parse-call + inbox + humanizer + gotchas
├── routines/             cloud routines (coworker, daily-review, self-audit, contact-migration)
└── memory/               runlog.md + audit.md (cursors live in Activity Log, not here)
```

## Routines (cloud)

| Name | Cadence | What |
|---|---|---|
| `coworker` | 3x weekdays | Tactical 24h sweep. New Fireflies transcripts → Activity Log + DEFCON Tasks + draft. Unanswered Gmail >24h → log + draft. Tomorrow's meetings → brief + prep task. |
| `daily-review` | 1x weekday evening | Rollup. Counterpart commitment verification. Deal property updates from today's accumulated Activity Log. Left-on-read prospecting. |
| `self-audit` | weekly Sundays 7pm | Drift patterns + stale Deal sweep. |
| `contact-migration` | manual one-shot | Notion Contacts → Google Contacts via Zapier. Requires Google Contacts enabled in Zapier MCP. |

## Skills

- `skills/parse-call.md` — Fireflies transcript → Activity Log + DEFCON Tasks + draft email + Deal updates. Called by `coworker` for each new transcript.
- `skills/humanizer.md` — voice rules (Hunter S. Thompson + Hemingway; banned em dashes + AI vocab + customer-service openers).
- `skills/gotchas.md` — canonical hard rules, status lifecycles, dedupe patterns, scope boundaries.

## Setup

1. Clone repo to wherever you want it locally (suggested: `~/Documents/Code/desk-monkey-routines`)
2. Authorize cloud connectors in your Anthropic account: Notion, Gmail, Google Calendar, Fireflies, Zapier
3. In the Zapier MCP config, enable these actions only (everything else duplicates direct MCP capability):
   - Gmail: Send Email, Archive Email, Move to Trash, Mark Read/Unread, Add Label to Email, Remove Label
   - Google Contacts: Create Contact, Find Contact, Update Contact (NOT YET ENABLED — needed for `contact-migration`)
   - Google Drive: Share File, Delete File (only if needed)
4. Create cloud routines in the Anthropic routine UI:
   - `coworker` → schedule `0 7,12,18 * * 1-5` → repo=this one → prompt: `Read routines/coworker/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `daily-review` → schedule `30 18 * * 1-5` → repo=this one → prompt: `Read routines/daily-review/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `self-audit` → schedule `0 19 * * 0` → repo=this one → prompt: `Read routines/self-audit/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `contact-migration` → no schedule (manual trigger) → prompt: `Read routines/contact-migration/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
5. First run: trigger `contact-migration` manually after Google Contacts is enabled in Zapier.

## Operating notes

- **Voice**: Hunter S. Thompson honesty meets Hemingway brevity. No em dashes. No AI vocab. See `skills/humanizer.md`.
- **Source of truth**: Reality (Gmail / Calendar / Fireflies) → Notion → repo memory files.
- **Never auto-send externally**: drafts only. Email goes to Gmail Drafts and Liam clicks Send.
- **Cron lives in the routine UI**: this repo doesn't execute schedules. The routine UI is canonical.
- **Notion is the database**: no Postgres, no state.json. Activity Log + DEFCON Tasks + Deals + Contacts + Projects handle everything.
- **Activity Log doubles as dedupe**: before processing a transcript or thread, query Activity Log for any row referencing the same source ID. If present, skip. Single source of truth for both data and idempotency.
- **Anti-dropped-ball**: every action item from a meeting / email / counterpart commitment becomes a DEFCON Task with Owner + Due + DEFCON 1-5. Counterpart-owned tasks (Owner=blank) get verified daily by daily-review.
