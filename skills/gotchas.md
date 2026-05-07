# Gotchas

Canonical hard rules + status lifecycles + dedupe + scope. Loaded by every routine.

## Hard NEVERs

- NEVER auto-send anything externally. Drafts only. Email = Gmail Drafts. Liam clicks Send.
- NEVER advance Deal Stage without explicit verbal commitment from the buyer in a transcript.
- NEVER touch Forecast Category. Liam's call.
- NEVER overwrite Deal Evolution. Always append, newest at top.
- NEVER dump raw transcripts into Attio Notes. Summaries only (2-3 lines).
- NEVER tag a non-Desk-Monkey thread (no matching Person/Deal Record in Attio) with a Deal. Skip entirely or attach a Note to a "Misc / Internal" placeholder Person Record.
- NEVER process the same Fireflies transcript twice. Attio Notes title-prefix dedupe gates this: query Notes attached to the matching Deal for a title starting with `MTG-<fireflies_id>` before creating.
- NEVER create duplicate Attio Notes. Title-prefix source ID is the dedupe key.
- NEVER create duplicate Attio Tasks. Dedupe by Deal + title prefix before creating.
- NEVER write to Notion CRM DBs (Contacts/Deals/Activity Log/DEFCON Tasks) from a routine. Those are legacy, archived after migration. Attio is canonical now.
- NEVER overwrite a populated Attio attribute with empty data on the migration. Only fill blanks.

## Attio tool-contract NEVERs

- NEVER hardcode Attio runtime tool names like `mcp__Attio__create_record`, `mcp__Attio__find_record`, or `mcp__Attio__update_record_attributes` in prompts. The Cloud Routine runtime decides which connector tool fulfills a capability. Use capability language (search records, list records, create record, upsert record, update record, list-attribute-definitions, create-note, list-tasks, create-task, update-task, list-lists, add-record-to-list).
- NEVER use fake Attio tools from old examples. `assert_record`, `update_record_attributes`, `find_record`, `find_list_entry`, `create_list_entry` are not tools the hosted MCP exposes. Do not write prompts that depend on them.
- NEVER use `assert_record` unless the live connector actually exposes that exact tool (it does not, at time of writing).
- NEVER call a Deal write before calling `list-attribute-definitions` for object `deals`. The workspace has custom fields and select options the routine cannot guess.
- NEVER call a list-entry write before calling `list-list-attribute-definitions` for the target list.
- ALWAYS prefer `upsert-record` when present. Otherwise: search-records → create-record / update-record. The upsert path is preferred because it's atomic and idempotent.
- NEVER auto-create a Deal below 0.80 confidence. Create a Liam-owned `[Pipeline Build]` review Task instead. Routines run unattended without approval prompts; the confidence gate is the only safeguard against pipeline pollution.
- NEVER skip the Attio runtime preflight in a routine that touches Attio. The `TOOL_CONTRACT` line in `memory/runlog.md` is the audit trail.
- If direct Attio MCP lacks a needed capability, use Zapier fallback ONLY for the actions allowed in `skills/attio-tooling.md`, AND only after logging `BLOCKED_TOOL_GAP`.
- BROWSER AUTOMATION IS FORBIDDEN for Attio CRM writes. Fragile, unsafe in unattended cloud routines, hard to dedupe.

## Attio Note title conventions

Every Note attached to a Deal Record has a title-prefix for dedupe + browseability:

- `MTG-<fireflies_id> — <meeting title>` — meeting from Fireflies
- `EMAIL-<gmail_thread_id> — <subject>` — email thread snapshot
- `BRIEF-<calendar_event_id> — <meeting title>` — pre-meeting prep brief
- `CALL-<YYYY-MM-DD> — <topic>` — manual phone call log
- `NOTE-<YYYY-MM-DD> — <topic>` — manual general note
- `LEGACY-<notion_page_id_short> — <activity>` — migrated from Notion Activity Log
- `MIGRATION-COMPLETE-<date>` — migration tracker marker

## Attio Task title conventions

Title prefix encodes DEFCON + Category + Owner since Attio Tasks don't expose custom attributes via the API:

`[DEFCON <1-5>] [<Category>] [<Owner>] <action description>`

- DEFCON: 1 (drop everything) → 5 (nice to have)
- Category: Apollo Setup / Pipeline Build / Email Infrastructure / Client Deliverable / Data Cleanup / Meeting Prep / Follow-up / Internal
- Owner: `Liam` if Liam-owned (assignees=[liam@...]); `<counterpart name>` if counterpart-owned (assignees=[])

Status semantics in title (when state is needed beyond is_completed):
- Append `[BLOCKED]` to content if waiting on someone (with verification path)
- Append `[KILLED]` and set is_completed=true if decided not to do

## DEFCON levels

- **DEFCON 1** → drop everything. Same-day. Compelling event triggered.
- **DEFCON 2** → today or tomorrow. Active deal in late stage.
- **DEFCON 3** → this week. Standard follow-up cadence.
- **DEFCON 4** → next week or two. Maintenance, slow-moving deal.
- **DEFCON 5** → nice to have. Backlog.

## Counterpart-owned task pattern

**Counterpart-owned tasks do NOT live in Attio.** Attio Tasks are exclusively Liam's todos. Counterparts have their own systems:

- **Miles (HFP USA)** → Apollo (his daily driver CRM)
- **Craig (CIO Tech)** → his own tracking; surfaced in shared Notion mutual action plans
- **Other counterparts** → handled in their own workspace

**Where counterpart commitments DO live:**

1. The Deal's Notion `MAP Draft` field — single table, owner column captures Liam vs counterpart, by-when, verification path
2. Attio Notes attached to the Deal — written into the action items section of the meeting Note, but NOT promoted to a Task

**For nudging when a counterpart is overdue:** `daily-review` reads the MAP, checks Liam's Gmail/Calendar for evidence the counterpart did the thing, and only creates a **Liam-owned** Attio Task ("[DEFCON 3] [Follow-up] [Liam] Nudge Craig on the new sending domain (overdue 2d)") when evidence is missing past the deadline.

Old pattern (deprecated): empty-assignee Attio Tasks tagged with the counterpart's name in the title prefix. That cluttered Liam's task list with rows he can't directly act on.

## Calendar invite authorization

Default: NEVER auto-create a Calendar event with attendees. CLAUDE.md hard NEVER stands.

**Exception:** when Liam explicitly authorizes a specific invite in conversation ("send the invite for X", "set up the meeting for tomorrow at 2pm"), the routine MAY call `create_event` with `notificationLevel=ALL` so attendees actually get the invite email. Authorization is **per-invite, not blanket** — Liam re-authorizes each one.

Email send (Gmail send action) is still NEVER. Drafts only, regardless of authorization on Calendar.

## Dedupe rules

- **Attio Notes**: title-prefix source ID. Always use the Attio list-notes capability for the parent Deal Record before creating; if title prefix matches, skip.
- **Attio Tasks**: title-prefix Deal + Category + Owner + action keywords. Use the Attio list-tasks capability for this Deal; if a similar Open task exists, append to that Task's content via update-task rather than creating a duplicate.
- **Audit**: never re-audit a week already in audit.md. Check headers first.
- **Migration**: never run twice. Attio Migration Tracker Note marker gates this.

## Scope (what's in vs out)

- **coworker** = Fireflies + Gmail + Calendar sweep, Attio Notes + Tasks + Deal updates + first-pass Gmail drafts + inbox noise classification. NOT iMessage. NOT Slack. NOT outbound Apollo sequences (yet).
- **daily-review** = evening rollup. Prunes today's first-pass meeting drafts. Verifies counterpart commitments (Attio Task sweep). Rolls Deal property updates across today's Notes. Left-on-read prospecting.
- **self-audit** = bounded read of last 200 lines of runlog + stale-deal sweep on Attio. Never reads the whole runlog.
- **contact-migration** = one-shot Notion CRM → Attio. NOT a recurring sync. Idempotent via Migration Tracker Note marker.

## Common drift to watch for

- Drafting in customer-service voice (em dashes, AI vocab, "circling back"). Read aloud test. See `skills/humanizer.md` for the 29 AI patterns.
- Updating Deal Stage on weak signals like "send me an email about it".
- Creating duplicate Attio Notes for the same Fireflies transcript or Gmail thread (forgetting the title-prefix dedupe).
- Creating duplicate Attio Tasks (forgetting to query existing tasks first).
- Forgetting to embed source ID in Note title (next run reprocesses everything because dedupe query can't match).
- Letting `[IN_PROGRESS]` runlog stubs stay [IN_PROGRESS] because the routine errored mid-run. Wrap work in try/recover, write 🔴 Failed report on exception.
- Counterpart-owned tasks getting created in Attio at all — they belong in the counterpart's system (Apollo for Miles, etc.) and the Notion MAP Draft. Liam's Attio task list is exclusively his own.
- Writing to Notion Contacts / Deals / Activity Log / DEFCON Tasks DBs (those are legacy; write to Attio).

## Non-Desk-Monkey filter

If a thread, transcript, or calendar event doesn't involve a Person Record or Deal Record in Attio, it's not Desk Monkey work. Don't try to route it.

A meeting/email/transcript counts as Desk Monkey work if at least one external attendee or sender matches:
- A Person Record in Attio (resolved via the Attio search-records capability filtered by email_addresses)
- Linked to an active Deal Record (Stage NOT IN [Closed Won, Closed Lost, Nurture, On Hold, Unresponsive])

If neither matches: skip entirely if low-signal, or attach a Note to a designated "Misc / Internal" Person Record if there's something worth keeping.

Examples that get skipped:
- Email noise from any non-Person-Record domain
- Internal coordination calls with people not in Attio People
- Generic notifications, newsletters, system mail (handled by coworker Step 2a noise classification)
