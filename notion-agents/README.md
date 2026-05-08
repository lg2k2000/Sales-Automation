# Notion AI Agents Skill

A Claude Skill for building six Notion 3.0 AI Agents in Liam Glennie's workspace. Notion can't self-build its own agents — that's why this Skill exists. Everything else (CRM databases, schemas, transcripts, skill templates) is already in the workspace.

## What this Skill builds

Six AI Agents that run inside Notion 3.0:

| # | Agent | Trigger |
|---|---|---|
| 1 | 🔗 Meeting Linker | scheduled hourly |
| 2 | 📧 Email Sweeper | scheduled hourly |
| 3 | 📅 Calendar Sweeper | scheduled 4h |
| 4 | 🦧 Triage / Daily Brief | scheduled 7am + 8pm MT |
| 5 | ✉️ Reply Executor | on-comment |
| 6 | 🧹 Inbox Sanitation | scheduled 4h |

## What this Skill does NOT do

- Build any database (Deals, Contacts, Activity Log, Projects already exist)
- Modify any database schema
- Migrate data from anywhere
- Send any email
- Write any code outside Notion's agent runtime

## File index (load all six)

| File | What's in it |
|---|---|
| `phased-plan.md` | The 4-phase build sequence (confirm → connections → deploy 6 agents → verify). |
| `agent-specs.md` | The 6 agents in detail: name, job, inputs, outputs, trigger, scope, tools, instructions block, test case. |
| `voice-rules.md` | Style guide that goes INTO every agent's instructions for content drafting. |
| `banned-list.md` | Hard no-fly list (vocab, phrases, punctuation, patterns) that goes INTO every agent's instructions. |
| `reference-ids.md` | Workspace UUIDs, DB IDs, contact IDs, signature block. |
| `operating-constraints.md` | Hard NEVERs, idempotency rules, rate limits, allowlists, status lifecycles. Goes INTO every agent's instructions. |

## How to use

**If you're a browser-control agent (Claude Computer Use / Claude Chrome / Comet) running this Skill:** load all six files. Start at Phase 0 in `phased-plan.md`. Confirm decisions with Liam before crossing each phase boundary.

**If you're a researcher comparing tools:** `phased-plan.md` and `agent-specs.md` together describe the job. Use them to evaluate which automation tool can actually do this in 2026.

## Prerequisites (already done; verify before starting)

- Notion workspace `Liam Glennie's Workspace HQ` exists with the 5 CRM databases populated
- Notion 3.0 plan with AI Agents enabled
- Notion Connections for Gmail + Google Calendar + Google Drive set up (Phase 1 verifies)
- Native Notion meeting transcription is on (transcripts arrive automatically)

## Open decisions (default in parens — confirm at Phase 0)

1. **Email + Calendar feed cadence** (hourly scheduled) — alternative: real-time webhook
2. **Daily Brief shape** (single overwriting page) — alternative: Briefs DB with one row per brief
