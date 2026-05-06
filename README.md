# Desk Monkey Routines

Sales-automation orchestration for Liam Glennie. **Attio is canonical for all customer relationship data.** Cloud routines (Anthropic Claude Code on the web) handle Fireflies + Gmail + Calendar sweeps and write to Attio. Notion holds projects + knowledge only. The repo is the operational scratchpad.

See `architecture.md` for the data flow diagram and `CLAUDE.md` for working memory loaded into every session.

## Layout

```
.
├── CLAUDE.md             auto-loaded into every session
├── architecture.md       data flow + tool layer
├── decisions.md          design-decisions log
├── skills/               parse-call + inbox + humanizer + gotchas
├── routines/             cloud routines (coworker, daily-review, self-audit, contact-migration)
└── memory/               runlog.md + audit.md (cursors live in Attio Notes title-prefix dedupe, not here)
```

## Routines (cloud)

| Name | Cadence | What |
|---|---|---|
| `coworker` | 2x weekdays | Tactical 24h sweep. New Fireflies transcripts → Attio Note + Tasks + Deal Record updates + first-pass Gmail draft. Unanswered Gmail >24h → Attio Note + draft. Tomorrow's meetings → Brief Note + prep Task. |
| `daily-review` | 1x weekday evening | Rollup. Prune today's first-pass meeting drafts (anti-AI-pacing). Counterpart commitment verification. Deal Record rollups from today's Notes. Left-on-read prospecting. |
| `self-audit` | weekly Sundays 7pm | Drift patterns + stale Deal sweep on Attio. |
| `contact-migration` | manual one-shot | Notion CRM (Contacts + Deals + Activity Log + DEFCON Tasks) → Attio. Idempotent. |

## Skills

- `skills/parse-call.md` — Fireflies transcript → Attio Note + Tasks + Deal Record updates + draft email. Called by `coworker` for each new transcript.
- `skills/inbox.md` — Gmail label scheme (timeless, role-based) + after-action playbook + label-by-Attio-Stage logic.
- `skills/humanizer.md` — voice rules (29 AI patterns to scrub from Wikipedia's Signs of AI Writing) + bad/good examples + signature spec.
- `skills/gotchas.md` — canonical hard rules, Attio Note + Task title conventions, dedupe patterns, scope boundaries.

## Setup

1. **Stand up Attio workspace.** Sign up for Attio (Free or Plus tier $34/seat to start). Create custom Deal attributes per CLAUDE.md "Required Attio Deal custom attributes" section. Create a "Migration Tracker" Person Record for the migration idempotency marker.
2. **OAuth Attio in Zapier MCP** at https://mcp.zapier.com/mcp/servers/<your-server-id>/app-auth/AttioCLIAPI (or use a direct Attio MCP if available).
3. **Authorize cloud connectors** in your Anthropic account: Notion (still needed for Projects + knowledge), Gmail, Google Calendar, Fireflies, Zapier, Attio.
4. **Zapier actions to enable** (only the gaps direct MCPs don't cover):
   - Gmail: Send Email, Archive Email, Move to Trash, Mark Read/Unread, Add Label to Email, Remove Label
   - Google Drive: Share File, Delete File (only if needed)
5. **Create cloud routines** in the Anthropic routine UI:
   - `coworker` → schedule `0 8,16 * * 1-5` → repo=this one → prompt: `Read routines/coworker/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `daily-review` → schedule `30 18 * * 1-5` → repo=this one → prompt: `Read routines/daily-review/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `self-audit` → schedule `0 19 * * 0` → repo=this one → prompt: `Read routines/self-audit/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `contact-migration` → no schedule (manual trigger) → prompt: `Read routines/contact-migration/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
6. **First run**: trigger `contact-migration` manually after Attio is set up. This pulls all Notion CRM data into Attio. After it completes, archive the Notion CRM DBs (Contacts/Deals/Activity Log/DEFCON Tasks) manually in the Notion UI.

## Operating notes

- **Voice**: Hunter S. Thompson honesty meets Hemingway brevity AND polite AND scrubbed of all 29 AI-writing tells. See `skills/humanizer.md`.
- **Source of truth**: Reality (Gmail / Calendar / Fireflies) → Attio (CRM canonical) → Notion (projects + knowledge) → repo memory files.
- **Never auto-send externally**: drafts only. Email goes to Gmail Drafts and Liam clicks Send.
- **Cron lives in the routine UI**: this repo doesn't execute schedules. The routine UI is canonical.
- **Attio is the database**: no Postgres, no state.json. Attio People + Companies + Deals + Notes + Tasks + Lists handle everything CRM-related.
- **Notion is for projects + knowledge only**: post-Closed-Won implementation tracking, methodology docs, SOPs, ideas. Not transactional CRM data.
- **Attio Notes title-prefix doubles as dedupe**: before processing a transcript or thread, query Notes attached to the Deal for one with title starting with the source ID prefix (`MTG-<fireflies_id>`, `EMAIL-<thread_id>`, `BRIEF-<calendar_event_id>`). If present, skip.
- **Anti-dropped-ball**: every action item from a meeting / email / counterpart commitment becomes an Attio Task. Title prefix encodes `[DEFCON <1-5>] [<Category>] [<Owner>]`. Counterpart-owned tasks (assignees=[]) get verified daily by daily-review.
- **Always commit to main**: no session branches. Cloud routines push to main directly.
