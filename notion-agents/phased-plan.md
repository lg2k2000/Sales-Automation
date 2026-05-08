# Phased plan

Build six Notion 3.0 AI Agents in Liam Glennie's workspace. Notion can't self-build agents — that's why this Skill exists. Everything else (databases, schemas, transcripts, skill templates) is already in Notion.

Pair with: `voice-rules.md`, `banned-list.md`, `reference-ids.md`, `agent-specs.md`, `operating-constraints.md`.

## Phase 0 — Confirm with Liam (5 min)

Before touching anything:

1. **Notion 3.0 AI Agents access verified.** Open Notion sidebar, look for the Agents entry. If missing, ask Liam (plan upgrade may be needed).
2. **Connections present.** Settings → Connections. Confirm Gmail + Google Calendar + Google Drive are connected. If any missing, walk Liam through OAuth (he must click Allow).
3. **Email + Calendar feed cadence.** Default: hourly scheduled sweep. Confirm or swap to webhook real-time.
4. **Daily Brief shape.** Default: a single overwriting page (`🦧 Daily Brief`) rebuilt twice daily. Confirm or swap to a Briefs DB.

**"Go" criteria:** Liam's answers written down before starting Phase 1.

## Phase 1 — Verify Connections + scopes

In Notion Settings → Connections, verify:

| Connection | Required scopes |
|---|---|
| Gmail (liam@deskmonkeyai.com) | gmail.readonly, gmail.modify, gmail.compose, gmail.send |
| Google Calendar (primary) | calendar.readonly, calendar.events |
| Google Drive | drive.readonly, drive.file |

If anything missing or scope-restricted, fix before deploying agents. Agents will fail silently otherwise.

**"Go" criteria:** every required scope on every connection shows green.

## Phase 2 — Deploy six AI Agents

Build in dependency order. Each agent: open Notion's Agents UI, create new agent, paste in name + description + instructions exactly per `agent-specs.md`, set Scope (which DBs/pages), set Tools (which Connections), set Trigger. Save. Manually trigger once with the test input. Verify expected output. Move on.

| # | Agent | Trigger | Why this order |
|---|---|---|---|
| 1 | 🔗 Meeting Linker | scheduled hourly | nothing depends on it; safe to test first |
| 2 | 📧 Email Sweeper | scheduled hourly | populates Activity Log for downstream views |
| 3 | 📅 Calendar Sweeper | scheduled 4h | adds BRIEF rows to Activity Log |
| 4 | 🦧 Triage / Daily Brief | scheduled 7am + 8pm MT | needs Activity Log rows from 1-3 |
| 5 | ✉️ Reply Executor | on-comment on Daily Brief | needs Daily Brief to exist |
| 6 | 🧹 Inbox Sanitation | scheduled 4h | independent; can run last |

**"Go" criteria per agent:**
1. Meeting Linker: trigger on May 6 "CRM and Sales Workflow Discussion" note (`3581acdf-10d2-80a0-947e-fde39dcc209d`) → links to HFP Deal, renames title, creates Activity Log row, drafts follow-up email in Gmail Drafts.
2. Email Sweeper: trigger → Activity Log rows for any threads from Contacts since the last sweep.
3. Calendar Sweeper: trigger → BRIEF rows for upcoming external meetings.
4. Triage: trigger → 🦧 Daily Brief page renders with sections, items numbered.
5. Reply Executor: post `skip 1` comment on Daily Brief → confirmation reply within 15 minutes.
6. Inbox Sanitation: trigger → promo deletion log entries with allowlist respected.

## Phase 3 — End-to-end verification

Run all six on real data, watching for failures.

1. Meeting Linker on real meeting note
2. Email Sweeper on past 4h of Gmail
3. Calendar Sweeper on next 24h of Calendar
4. Triage builds Daily Brief
5. Liam comments on Daily Brief, Reply Executor responds
6. Inbox Sanitation runs cleanup

**"Go" criteria:** all 6 produce expected output AND scheduled triggers fire on their own within their first cycle (so 1-4 hour wait before declaring done).

If anything fails: screenshot, name what tried, name what's confusing, ask Liam.

## Stop conditions (any phase)

- A required Connection has expired tokens
- A required Notion DB / page is missing or trashed
- Notion Agents UI changed and the steps no longer match — screenshot + ask Liam
- Liam comments "stop" or "wait" on any progress update
