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
| `connector-diagnostic` | manual, before production runs | Read-only. Confirms Cloud Routine sees the same Attio tools as a normal Claude chat. Logs tool contract. |
| `contact-migration` | manual one-shot | Notion CRM (Contacts + Deals + Activity Log + DEFCON Tasks) → Attio. Idempotent. |

## Skills

- `skills/attio-tooling.md` — canonical Attio tool contract: runtime preflight, capability names (no hardcoded MCP function names), Deal creation/update protocols, Zapier fallback rules. Read by every routine that touches Attio.
- `skills/parse-call.md` — Fireflies transcript → Attio Note + Tasks + Deal Record updates + draft email. Called by `coworker` for each new transcript.
- `skills/inbox.md` — Gmail label scheme (timeless, role-based) + after-action playbook + label-by-Attio-Stage logic.
- `skills/humanizer.md` — voice rules (29 AI patterns to scrub from Wikipedia's Signs of AI Writing) + bad/good examples + signature spec.
- `skills/gotchas.md` — canonical hard rules, Attio Note + Task title conventions, dedupe patterns, scope boundaries.

## Setup

1. **Stand up Attio workspace.** Sign up for Attio (Free or Plus tier $34/seat to start). Create custom Deal attributes per CLAUDE.md "Required Attio Deal custom attributes" section. Create a "Migration Tracker" Person Record for the migration idempotency marker.
2. **Add Attio hosted MCP in Claude:**
   - Settings / Customize → Connectors
   - Browse connectors → Attio, or add custom connector URL: `https://mcp.attio.com/mcp`
   - Complete OAuth
3. **In each Claude Routine, open connector settings and confirm Attio is included.** Routines see a different connector surface than normal chats; this step is mandatory before scheduling.
4. **Keep Zapier MCP connected** only for fallback actions and Gmail operations that the direct Gmail connector lacks:
   - Gmail: Send Email, Archive Email, Move to Trash, Mark Read/Unread, Add Label to Email, Remove Label
   - Google Drive: Share File, Delete File (only if needed)
   - Attio fallback writes (only if a needed action is missing from the direct Attio MCP)
5. **Authorize cloud connectors** in your Anthropic account: Attio (primary CRM), Notion (Projects + knowledge), Gmail, Google Calendar, Fireflies, Zapier (fallback only).
6. **Run connector diagnostic before production runs.** Trigger `connector-diagnostic` manually from the Anthropic routine UI. It is read-only. It logs which Attio capabilities the Cloud Routine actually sees. If a required capability is missing for `contact-migration` or `coworker`, fix the connector setup before scheduling.
7. **Create cloud routines** in the Anthropic routine UI:
   - `connector-diagnostic` → no schedule (manual) → prompt: `Read routines/connector-diagnostic/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `coworker` → schedule `0 8,16 * * 1-5` → repo=this one → prompt: `Read routines/coworker/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `daily-review` → schedule `30 18 * * 1-5` → repo=this one → prompt: `Read routines/daily-review/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `self-audit` → schedule `0 19 * * 0` → repo=this one → prompt: `Read routines/self-audit/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `contact-migration` → no schedule (manual trigger) → prompt: `Read routines/contact-migration/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
8. **First production run**: after connector diagnostic confirms Attio capabilities are present, trigger `contact-migration` manually. This pulls all Notion CRM data into Attio. After it completes, archive the Notion CRM DBs (Contacts/Deals/Activity Log/DEFCON Tasks) manually in the Notion UI.

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
