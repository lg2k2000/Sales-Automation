# SKILL: Build Notion AI Agents

**You are Claude operating a browser or full desktop on Liam Glennie's machine.** Your job is to build six Notion 3.0 AI Agents inside Liam's Notion workspace. Notion has no API for agent CRUD — the only way to create them is by clicking through Notion's UI. That's why you exist.

## Recommended runtime

**Anthropic Claude Computer Use on macOS (strongest)** — see the full desktop, switch between Notion app + Chrome (for any web auth) + Mail (for 2FA codes). Use the Notion desktop app for full keyboard-shortcut support.

Alternatives that work (with reduced reliability):
- **Claude for Chrome** (browser extension) — Notion in browser at https://www.notion.so. Good for the agent build itself, weaker on OAuth handoffs.
- **Perplexity Comet** — same as Claude for Chrome category. Voice rule adherence is weaker.

## What you're building

Six agents that run inside Notion 3.0:

| # | Agent | Trigger | Spec file section |
|---|---|---|---|
| 1 | 🔗 Meeting Linker | scheduled hourly | `agent-specs.md` §1 |
| 2 | 📧 Email Sweeper | scheduled hourly | `agent-specs.md` §2 |
| 3 | 📅 Calendar Sweeper | scheduled 4h | `agent-specs.md` §3 |
| 4 | 🦧 Triage / Daily Brief | scheduled 7am + 8pm MT | `agent-specs.md` §4 |
| 5 | ✉️ Reply Executor | on-comment on Daily Brief | `agent-specs.md` §5 |
| 6 | 🧹 Inbox Sanitation | scheduled 4h | `agent-specs.md` §6 |

## What you do NOT do

- Don't modify any database schema (already built; verify only)
- Don't migrate any data (already done by Liam)
- Don't write any code outside Notion's agent runtime
- Don't send any email yourself (drafts only, and only via the Reply Executor agent you build)
- Don't create Calendar invites yourself

## Files to load

All in this directory. Load them in this order at start:

1. `SKILL.md` (this file) — what to do
2. `phased-plan.md` — the 4 phases with go/no-go gates
3. `agent-specs.md` — full spec for each of the 6 agents (paste the Instructions block verbatim into the Notion Agent UI)
4. `voice-rules.md` — paste into every agent's Instructions where the agent drafts content
5. `banned-list.md` — same
6. `operating-constraints.md` — paste into every agent's Instructions
7. `reference-ids.md` — workspace UUIDs, DB IDs, contact IDs

## Pre-flight checklist (do all of these before clicking anything)

- [ ] You're signed into Notion as liam@deskmonkeyai.com
- [ ] Workspace is `Liam Glennie's Workspace HQ`
- [ ] You can see the AI Agents entry in the left sidebar (or in Settings → AI Agents). If missing, **STOP** and ask Liam.
- [ ] Settings → Connections shows Gmail, Google Calendar, Google Drive all connected with green status. If any missing, **STOP** and walk Liam through OAuth — he must click Allow on Google's screen.
- [ ] You've read the Hard NEVERs in `operating-constraints.md`.
- [ ] You've read the voice rules in `voice-rules.md` and the banned list in `banned-list.md`.

## How to create one agent in Notion (the UI flow)

The Notion 3.0 Agent UI may have changed since this Skill was written. Adapt as needed; the conceptual steps stay the same.

1. **Open the AI Agents UI.** Sidebar left, click "AI Agents". (Alt path: Settings & members → AI Agents.)
2. **Click "+ New Agent"** (or "Create agent").
3. **Name** field: paste the agent name from `agent-specs.md` (e.g. `🔗 Meeting Linker`).
4. **Description** field: paste the one-line job description.
5. **Instructions** field (large textarea): paste the Instructions block from `agent-specs.md` for this agent. Then append:
   - The full text of `voice-rules.md` under a heading `## Voice rules`
   - The full text of `banned-list.md` under a heading `## Banned list`
   - The full text of `operating-constraints.md` under a heading `## Operating constraints`
   - The Reference IDs section relevant to this agent from `reference-ids.md` under a heading `## Reference IDs`
6. **Scope** (which databases / pages the agent can read+write): set per `agent-specs.md` for this agent. Use the data source IDs from `reference-ids.md`.
7. **Tools** / **Connections** (which external tools this agent can call): set per `agent-specs.md`. Toggle ON only what's listed; leave everything else OFF.
8. **Trigger**: set per `agent-specs.md`. For scheduled, use Notion's cron-style picker. For on-comment / on-row-create, find the corresponding option (UI may label these "Event triggers" or "Automations").
9. **Save**.
10. **Test** by manually triggering once with the test input listed in `agent-specs.md` for this agent. Verify the expected output. If it fails, screenshot and ask Liam before moving on.

Repeat for all 6 agents in the order listed in `phased-plan.md` Phase 2.

## When to stop and ask Liam

Halt and ping Liam (via comment on the page you're working on, or in chat) before:

- Any **destructive action** (deleting a page, dropping a property, archiving a Deal, anything that loses data)
- Any **OAuth flow** (Liam clicks Allow; you wait)
- Any **UI element you don't recognize** (Notion may have changed; screenshot + ask)
- Any **failed test** at a Phase boundary
- Any **rate limit / API error** that recurs

## Style of communication with Liam during the build

- Match the rules in `voice-rules.md` and `banned-list.md`. Yes, they apply to what you write to Liam too.
- Direct, concrete, no filler. Peer asking peer for help.
- When stuck: paste the screenshot, name what you tried, name what's confusing, ask one question.
- After each phase boundary: post a 3-6 line summary of what you changed and what's next. Wait for "go" before crossing.

## Final acceptance

When all 6 agents are deployed, scoped, triggered, and verified:

1. Post a one-paragraph summary to Liam:
   - 6 agents deployed (list names + triggers)
   - Test results per agent
   - Any drift / blockers logged
   - Next pickup: when scheduled triggers will fire on their own
2. Open the Daily Brief page (`🦧 Daily Brief`) and confirm it renders. This is the surface Liam will use going forward.
3. Tell Liam: "Build complete. Daily Brief is live. Comment on items to act."
