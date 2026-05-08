# Operating constraints

What every agent must respect, what requires Liam approval, what's never allowed. Read first by any agent build or any tool driving the build.

## Hard NEVERs (no agent ever does these autonomously)

- **NEVER auto-send Gmail externally.** Drafts only. Send only on explicit Liam comment `send <N>` for that specific item.
- **NEVER auto-create Calendar events with external attendees.** Send invite only on explicit Liam comment `send invite <N>` (with proposed window). Per-invite authorization, not blanket.
- **NEVER advance Deal Stage.** Even on `advance N` from Liam — write a readiness flag in Deal Evolution; Liam advances manually in the UI.
- **NEVER modify Forecast Category.** Liam's call exclusively.
- **NEVER overwrite Deal Evolution.** Always append. Newest entry at top. Erasing history is forbidden.
- **NEVER dump raw transcripts into Activity Log Summary.** 2-3 line digests in Liam's voice only. Raw content goes in `Raw Content` expanded property.
- **NEVER process the same source twice.** Title-prefix dedupe (`MTG-`, `EMAIL-`, `BRIEF-`, `CALL-`, `NOTE-`) is the gate. Search before write.
- **NEVER tag a non-Desk-Monkey thread with a Deal.** If sender / attendee doesn't match a Contact in an active Deal, skip or route to "Misc / Internal" placeholder Contact.
- **NEVER create duplicate Tasks.** Dedupe by Deal + title prefix (`[DEFCON N] [Category] [Owner]`) before creating.
- **NEVER create counterpart-owned Tasks.** Counterpart commitments live in the meeting Note's Action Items + the Deal's MAP Draft. Liam's task list stays exclusively his own. Build a Liam-owned nudge task only when evidence of counterpart completion is missing past their deadline.
- **NEVER skip the Daily Brief.** Even when nothing surfaces, post "✅ Nothing to surface. Inbox clean. Next meeting at <time>." Silent failure is worse than empty noise.
- **NEVER delete receipts or billing email.** Receipts allowlist is sacrosanct (tax records).
- **NEVER delete email from a sender on an active (non-Closed) Deal.** Even if it looks promotional. Active deal contacts override sanitation rules.
- **NEVER skip Liam's voice rules.** Read `voice-rules.md` and `banned-list.md` before any external draft.
- **NEVER use browser automation to write to Notion if the API can do the job.** Notion's official API + MCP is the canonical write path. Browser is for agent UI configuration only.

## Destructive actions (require explicit Liam approval per action)

The agent or build tool stops, surfaces what it's about to do, and waits for "yes" before proceeding:

- Deleting a Notion page (any DB, any sub-page)
- Dropping a column / property from any DB schema
- Archiving a Deal or Project
- Deleting a Gmail draft (Liam can do this manually; don't ask agents)
- Sending email to an external recipient (per item)
- Creating a Calendar invite with attendees (per invite)
- Modifying / replacing the email signature block
- Force-pushing to git (if any agent ever touches the repo)
- Removing rows from the Run Log

## Idempotency rules

Every write must be safe to re-run. Use unique source-ID title prefixes:

| Source | Title prefix | Example |
|---|---|---|
| Notion meeting note | `MTG-<note_id_short>` | `MTG-3581acdf — May 6 — Houston Foam — Sales POC kickoff` |
| Gmail thread | `EMAIL-<thread_id>` | `EMAIL-19e02a408a9a0d64 — Tentative: Notion walkthrough` |
| Calendar event | `BRIEF-<event_id>` | `BRIEF-ei97ak4krvoui8bt353io0kkio — CIO Tech + Apollo` |
| Manual call log | `CALL-<YYYY-MM-DD>` | `CALL-2026-05-08 — Andrew BlueAlly walkthrough` |
| Manual note | `NOTE-<YYYY-MM-DD>` | `NOTE-2026-05-07 — Notion architecture decisions` |
| Migrated from legacy Notion | `LEGACY-<page_id_short>` | (one-shot only; not used post-migration) |
| Migration tracker | `MIGRATION-COMPLETE-<date>` | one-shot marker |

Before creating any row, query the parent's children for a row whose title starts with the source-ID prefix. If found: skip the create, optionally append to the existing row's body.

## Rate limits

- **Email Sweeper:** at most 100 threads per run. Larger inboxes paginate across runs.
- **Inbox Sanitation:** at most 50 deletions per run. Surfaces remainder in next Daily Brief.
- **Meeting Linker:** at most 20 meeting notes per run. Larger backlogs paginate.
- **All agents:** if Notion API rate limits hit, back off exponentially (2s, 4s, 8s, 16s) and retry up to 4 times. Then log and exit clean.
- **Gmail send:** send at most 10 emails per Reply Executor run. Larger sends require Liam to break them across replies.

## OAuth scopes (Notion Connections)

| Connection | Required scopes | Why |
|---|---|---|
| Gmail | gmail.readonly, gmail.modify, gmail.compose, gmail.send | Email Sweeper reads + labels; Reply Executor sends; Meeting Linker drafts |
| Google Calendar | calendar.readonly, calendar.events | Calendar Sweeper reads; Reply Executor creates events |
| Google Drive | drive.readonly, drive.file | SOW / attachment reading |
| Slack | channels:read, chat:write, users:read | optional sidecar surface; can omit if not using |

Liam clicks Allow on each Google consent screen. Agents cannot bypass that.

## Allowlists (never-delete senders)

**Receipts (NEVER delete, NEVER unsubscribe):**
- stripe.com, anthropic.com, notion.so, attio.com, apollo.io, slack.com, zapier.com, google.com (billing), microsoft.com (billing), aws.amazon.com (billing), linkedin.com (billing only)

**Active deal contacts (NEVER delete):**
- Any sender on a Contact linked to a non-Closed Deal. Re-checked on every Inbox Sanitation run from Contacts DB + Deals DB.

**Personal (NEVER touch):**
- Empty by default. Populate via `allowlist <sender>` Liam comment on Daily Brief.

## Stage / Status lifecycles

**Activity Log Status:**
- `Logged` — done, archived
- `Open Action` — needs Liam reply or action
- `Needs Routing` — agent low-confidence; Liam disambiguates
- `Send Now` — Reply Executor send-worker picks up within 15 min and sends. ONLY for Channel=Email or Text AND Direction=Out

**Deal Stage:**
- New / Discovery / Qualified / POC / Co-Building / Proposal / Negotiation / Procurement / Closed Won / Closed Lost / Nurture / On Hold / Unresponsive
- Advancement only via Liam-confirmed buyer commitment in transcript. Agents flag readiness via `[DEFCON 2] [Internal] [Liam] <Deal>: ready for <new stage> — <evidence>` task.

**Selling With qualification gates:**
- **Qualified gate:** 3+ Champion Tests Passed → flag for Stage advance. Don't flip stage automatically.
- **Co-Building gate:** Buyer-Owned Action Ratio ≥ 50% → flag for Stage advance.

## Voice gate

Every external draft (email body, Slack message, page content visible to non-Liam) passes this checklist before saving:

1. Read aloud test
2. No banned vocab (see `banned-list.md`)
3. No em dashes
4. No customer-service openers
5. No commanding-the-recipient phrasing
6. Email signature block on every email draft
7. Specific time windows offered for any scheduling proposal
8. Anti-AI pass: name 3-4 tells, fix them, re-read

If any check fails, redo. If any draft slips through and Liam catches an AI tell, the agent's voice gate is broken; flag in Run Log.

## Counterpart-owned task discipline

- Counterpart commitments live in the meeting Note Action Items + the Deal's MAP Draft.
- Never create Notion / DEFCON Tasks with empty assignees or counterpart-named-as-owner.
- Liam's task list stays his own. Period.
- A Liam-owned nudge task is created only when:
  - The MAP shows a counterpart commitment past deadline
  - Activity Log + Calendar show no evidence of completion
  - Title format: `[DEFCON 3] [Follow-up] [Liam] Nudge <counterpart> on <commitment> (overdue Xd)`

## Run Log

Every agent writes a row to a 📋 Run Log DB on every invocation, even on success or no-op. Schema (build in Phase 1):

| Field | Type | Notes |
|---|---|---|
| Run | title | `<Agent name> — <ISO timestamp>` |
| Status | select | Healthy / Drift / Failed / No-op |
| Agent | select | Meeting Linker / Email Sweeper / etc. |
| Trigger | select | Schedule / Manual / On-comment / On-row-create |
| Reads | text | counts: meetings=N, threads=N, events=N |
| Writes | text | counts: log_rows=N, drafts=N, briefs=N |
| Skipped | text | reasons: "MTG already exists for <id>", "no Deal match", etc. |
| Drift / Blockers | text | what went wrong; tool gaps; ambiguous intent |
| Next pickup | text | timestamp / state for the next run |
| Created at | created_time | auto |

The Run Log replaces the repo's `memory/runlog.md`. Audit-friendly, queryable, no git commits.

## Stop conditions for any agent

The agent halts and writes a Run Log row with Status=Drift if:

1. A required Notion DB / page is missing or trashed
2. A Connection (Gmail / Calendar) has expired tokens
3. A required attribute is renamed / dropped (schema drift)
4. Rate limit retries exhausted
5. An unrecognized field option appears in a select / status (`FIELD_OPTION_GAP`)
6. The agent's prompt produces low-confidence intent on a destructive action

The agent never silently fails. Every run produces a Run Log row.
