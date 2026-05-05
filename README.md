# Desk Monkey Routines

Sales-automation orchestration for Liam Glennie. Cloud routines (Anthropic Claude Code on the web) handle Fireflies + Gmail + Calendar + Notion sweeps. Local Mac tasks handle iMessage. The repo is the shared persistence layer between the two.

See `architecture.md` for the data flow diagram and `CLAUDE.md` for working memory loaded into every session.

## Layout

```
.
├── CLAUDE.md             auto-loaded into every session
├── architecture.md       data flow + tool layer diagram
├── .mcp.json             local Claude Code MCP config (Notion only; cloud connectors auth'd via routine UI)
├── skills/               humanizer voice rules + gotchas
├── routines/             cloud routines (coworker, contact-migration, self-audit)
├── local-tasks/          Mac-bound tasks (triage, send-worker)
└── memory/               runlog (append-only), audit (weekly findings)
                          NOTE: cursors live in Notion Skill State DB, not here
```

## Routines (cloud)

| Name | Cadence | What it does |
|---|---|---|
| `coworker` | 3x weekdays (set in routine UI) | Sweep Fireflies, Gmail, Calendar. Log to Notion, draft follow-ups, update Deal pages. |
| `contact-migration` | one-shot manual | Pull Notion Contacts → create Google Contacts via Zapier. Idempotent. |
| `self-audit` | weekly Sundays 7pm | Scan runlog tail, write findings to `memory/audit.md`. |

## Local tasks (Mac-only)

| Name | Cadence | Why local |
|---|---|---|
| `triage` | hourly 8-18 | iMessage MCP is stdio-only |
| `send-worker` | every 15 min weekdays 8-18 | iMessage send is stdio-only |

## Setup

1. Clone repo to wherever you want it locally (suggested: `~/Documents/Code/desk-monkey-routines`)
2. Authorize cloud connectors in your Anthropic account: Notion, Gmail, Google Calendar, Fireflies, Zapier
3. In the Zapier MCP config, enable these actions only (everything else duplicates direct MCP capability):
   - Gmail: Send Email, Archive Email, Move to Trash, Mark Read/Unread, Add Label to Email, Remove Label
   - Google Contacts: Create Contact, Find Contact, Update Contact
   - Google Drive: Share File, Delete File (only if you'll need them)
4. Create cloud routines at the Anthropic routine UI:
   - `coworker` → schedule `0 7,12,18 * * 1-5` (or your preferred 3x weekday cadence) → repo=this one → prompt: `Read routines/coworker/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `contact-migration` → no schedule (manual trigger) → repo=this one → prompt: `Read routines/contact-migration/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
   - `self-audit` → schedule `0 19 * * 0` → repo=this one → prompt: `Read routines/self-audit/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`
5. Migrate local tasks: paste your existing triage and send-worker SKILL.md content into `local-tasks/*/SKILL.md` (placeholders are in place with the repo-sync wrapper at the top).
6. First run: trigger `contact-migration` manually to populate Google Contacts.

## Operating notes

- **Voice**: Hunter S. Thompson honesty meets Hemingway brevity. No em dashes. No AI vocab. See `skills/humanizer.md`.
- **Source of truth**: Reality (iMessage / Gmail / Calendar / Fireflies) → Notion → repo memory files.
- **Never auto-send externally**: drafts only. Email goes in Gmail Drafts. iMessage waits on the Notion Send Now flag.
- **Cron lives in the routine UI**: this repo doesn't execute schedules. No GitHub Actions, no cron files. The routine UI is canonical.
- **Notion is the database**: no Postgres, no separate store. Activity Log + Contacts + Deals + DEFCON Tasks + Projects + Skill State collections handle everything.
- **Anti-dropped-ball**: every action item from a meeting / email / counterpart commitment becomes a DEFCON Task with Owner + Due + DEFCON 1-5 priority. Counterpart-owned tasks get verified daily by sweeping Gmail / Calendar for proof of completion.
- **State lives in Notion**: operational cursors (last_fireflies_id, last_gmail_sweep, etc.) live in the Skill State DB, not in repo files. This means no git-merge conflicts on cursors and Liam can see state in his dashboard.
