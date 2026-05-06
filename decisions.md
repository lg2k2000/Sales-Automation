# Design Decisions Log

Captures architectural decisions made during the build of Desk Monkey. Each entry: what got decided, why, what got rejected.

## Brain layer

### Attio is canonical CRM, not Notion (May 2026 pivot)
- **Decided:** All customer relationship data lives in Attio: People, Companies, Deals, Notes (= activity log), Tasks (= DEFCON action items), Lists (= saved segments). Notion's role narrows to Projects (post-Closed-Won) + knowledge base + SOPs.
- **Why:** Notion-as-CRM was always a stretch. Attio is purpose-built for relationship-driven sales: native email-thread tracking on contacts, auto-enrichment, real-time relationship graph, list segments, custom attributes on Deal objects (full Selling With schema). UX is obviously better than DIY Notion CRM. Built-in features replace the manual scaffolding we had.
- **Rejected:** Sticking with Notion as canonical (rejected after side-by-side comparison). HubSpot/Salesforce (overkill for this volume + worse UX). Postgres (still no, see earlier decision).
- **Migration:** one-shot `contact-migration` routine pulls Notion Contacts → Attio People, Notion Deals → Attio Deals, Notion Activity Log → Attio Notes, Notion DEFCON Tasks → Attio Tasks. Idempotent via a Migration Tracker Note marker. After migration, Notion CRM DBs archived manually.

### Attio Notes double as the dedupe ledger
- **Decided:** Before processing a transcript or thread, query Attio Notes attached to the matching Deal Record for one with a title starting with the source ID prefix (`MTG-<fireflies_id>`, `EMAIL-<thread_id>`, `BRIEF-<calendar_event_id>`). If present, skip.
- **Why:** Single source of truth for both data and idempotency. No `state.json`, no Skill State DB, no separate dedupe mechanism.
- **Note title convention:** `<TYPE>-<source_id> — <subject>`. The TYPE prefix tells the dedupe query what to match.

### Attio Tasks encode DEFCON in title prefix
- **Decided:** Attio Task content/title format: `[DEFCON <1-5>] [<Category>] [<Owner>] <action>`. Owner field = assignees=[liam@...] for Liam-owned, empty for counterpart-owned.
- **Why:** Attio Tasks don't expose custom attributes via the API (only built-in fields: content, deadline_at, assignees, is_completed, linked_records). Title-prefix encoding gives us DEFCON priority + Category + Owner without needing custom attributes.
- **Trade-off:** less queryable than custom attributes (filtering "all DEFCON 1 tasks" requires substring matching). Acceptable given the volume.

### Two cadences, four routines
- **Decided:** `coworker` (2x weekdays, tactical), `daily-review` (1x evening, rollup), `self-audit` (weekly), `contact-migration` (manual one-shot Notion → Attio).
- **Why:** Tactical vs strategic split keeps the fast loop fast. Counterpart verification + Deal property rollups need a calmer pass; tactical sweep can't wait on those.
- **Rejected:** Single mega-routine. Real-time event-driven (would require webhooks).

### Polling, not webhooks
- **Decided:** Routines pull from Fireflies / Gmail / Calendar on a schedule.
- **Why:** Webhooks add infra complexity for marginal latency improvement. Hours of latency is fine for follow-up work.

## Tool layer

### Direct MCPs primary, Zapier fills gaps
- **Decided:** Use direct MCPs for read AND write whenever they exist. Zapier only for actions direct MCPs don't expose.
- **Why:** Direct MCPs are faster, free per-call, more structured. Zapier costs Zap tasks per call and has less control over response shape.
- **Currently filling gaps via Zapier:** Gmail Send / Archive / Trash / Mark Read/Unread / Add Label / Remove Label. Attio fallback only when the direct hosted MCP is missing a needed action.

### Use Attio hosted MCP as primary CRM connector
- **Decided:** Direct Attio hosted MCP at `https://mcp.attio.com/mcp` is primary for CRM reads and writes. Zapier MCP is fallback only.
- **Why:** Attio publishes a hosted MCP server with OAuth and a documented capability surface (`search-records`, `list-records`, `get-records-by-ids`, `create-record`, `upsert-record`, `update-record`, `list-attribute-definitions`, `list-lists`, `list-list-attribute-definitions`, `list-records-in-list`, `add-record-to-list`, `create-note`, `list-tasks`, `create-task`, `update-task`). Direct MCP gives structured access without per-call Zap costs and without Zapier's response-shape limitations.
- **Repo prompts must not hardcode runtime tool-call names.** Cloud Routines may expose a different connector surface than a normal Claude chat. Names like `mcp__Attio__create_record`, `mcp__Attio__assert_record`, `mcp__Attio__update_record_attributes`, `mcp__Attio__find_record`, `mcp__Attio__create_list_entry` are not guaranteed at runtime. Prompts use capability language (e.g. "use the Attio create-note capability") and `skills/attio-tooling.md` runtime preflight maps capability → actual tool.
- **Runtime preflight is required.** Every routine that touches Attio inspects available connector tools, logs a `TOOL_CONTRACT` line to `memory/runlog.md`, and gracefully degrades on `BLOCKED_TOOL_GAP`. The `connector-diagnostic` routine validates this manually before production runs.
- **Deal creation uses `create-record` / `upsert-record` semantics** combined with `list-attribute-definitions` for object `deals`. No write happens before the schema lookup. Select-field values that don't match an allowed option are logged as `FIELD_OPTION_GAP` and skipped, not written incorrectly.
- **Pipeline list membership uses `add-record-to-list`** as a separate action from creating the Deal record itself. Adding to a list is preceded by `list-list-attribute-definitions` for the target list to discover required entry attributes.
- **Confidence gate on Deal creation.** Routines run unattended without approval prompts. Pipeline pollution is irreversible at scale. Only create a Deal at confidence ≥ 0.80 (real sales meeting, pricing/proposal, POC, migration, renewal pain, explicit next step). Below that, create a Liam-owned review Task.
- **Browser automation rejected for CRM writes.** Fragile, unsafe in unattended cloud routines, hard to dedupe. Direct MCP + Zapier fallback covers the surface area.
- **Rejected:** Hardcoding `mcp__Attio__*` names in prompts (silently breaks when the runtime exposes a different name); `assert_record` (not on the hosted MCP surface); browser automation (fragility, dedupe risk); making Zapier the primary Attio path (latency, cost, less structured).

### Gmail filter creation is not exposed; coworker does it
- **Decided:** `coworker` Step 2a sweeps inbox every coworker run and labels + archives system noise (newsletters, receipts, notifications) via Zapier `Add Label` and `Archive Email`.
- **Why:** Neither direct Gmail MCP nor Zapier expose `users.settings.filters.create`. Routine-based approximation is the next-best option.
- **Trade-off:** worst-case 8-hour delay (coworker runs 2x/day) before noise gets out of inbox.

### Fireflies for transcripts (not Attio Voice)
- **Decided:** Fireflies Pro plan ($18/mo). Bot joins meetings via calendar invite, transcribes regardless of platform.
- **Why:** Works as a guest on others' meetings (Google Meet recording requires host). Has direct MCP. Returns structured summaries + action items. Cheaper than Attio Voice (which is bundled in Attio Pro $69/seat).
- **Reassess:** if Attio Voice quality + workflow integration outweighs the $50/mo savings, switch later.

## Risk controls

### Drafts only, never autonomous sends
- **Decided:** Routines never call Zapier `Send Email`. Email drafts go to Gmail Drafts via direct `create_draft`. Liam clicks Send in either Gmail UI or Attio UI (Attio's Gmail integration shows the same drafts).
- **Why:** Single point of human review. Embarrassing-send risk concentrated in one click that Liam controls.
- **Calendar invites with attendees:** routines never auto-create. They surface an Attio Task; Liam creates the event manually.

### Wait-and-prune for meeting follow-ups
- **Decided:** `coworker` writes a first-pass draft within hours of meeting end. `daily-review` (evening) prunes it to 30-50% length and replaces the draft. Liam sees the pruned version next morning.
- **Why:** A draft sent 30 seconds after a meeting reads as AI. A tightened draft the next morning reads as a person.
- **Marker:** `[first-pass draft pending prune]` in the Attio Note content, replaced with `[draft pruned <ISO>]` after prune pass.

### Voice: polite AND not AI-polished, both at once
- **Decided:** Hunter S. Thompson honesty + Hemingway brevity, AND polite, AND scrubbed of all 29 AI-writing tells. Not either/or, not three-of-four. All four.
- **Why:** Brevity without politeness reads as bossy. Politeness without authenticity reads as a customer-service bot. The 29 AI-writing patterns from Wikipedia (em dashes, rule-of-three, AI vocab, signposting, hedging, etc.) all get scrubbed. See `skills/humanizer.md`.
- **The bar:** sounds like a peer texting a peer. Reads aloud naturally. Has soul (specific feelings, opinions, varied rhythm).

### Email signature mandatory
- **Decided:** Every email draft ends with the full Liam Glennie / phone / website signature block. First-name-only sign-offs not acceptable.
- **Why:** Looks professional, not like an AI bot impersonating Liam.

## Inbox strategy

### Role-based labels, not deal-based
- **Decided:** `Customer`, `Prospect`, `Partner`, `Vendor`, `Personal` plus state labels `_Action`, `_Waiting`, `_Reference` plus `System/Receipts`, `System/Newsletters`, `System/Notifications`. 11 labels total, all timeless.
- **Why:** Roles persist for years; deals come and go. Use Gmail search for company-specific finds.
- **Rejected:** One label per active Deal — accumulates cruft over time.

### Coworker auto-labels by relationship type, sourced from Attio Deal Stage
- **Decided:** When `coworker` processes a Gmail thread tied to an Attio Person, it applies the relationship label based on the Person's linked Deal Stage (Closed Won → Customer; active stages → Prospect; Referrer attribute → Partner) plus a state label (`_Action` or `_Waiting`).
- **Why:** Removes manual labeling from Liam's daily routine.

## What got rejected wholesale

### iMessage triage / send-worker (deferred, not in current build)
- **Decided not to wire up:** Mac stdio MCP for iMessage (mac_messages_mcp). Local cron-based triage and send-worker.
- **Why:** Liam decided not to set up the local Mac infrastructure for now. iMessage triage can return later if priorities change.

### Local Claude Code MCP config (`.mcp.json`)
- **Decided not to keep:** Liam isn't running Claude Code locally on this repo. The file added cognitive overhead with no benefit.

### Slack as inbound channel
- **Status:** Discussed, not implemented. Slack MCP via Zapier is doable (~30 min setup) but defer until Attio-as-inbox proves insufficient.

### Skill State + Skills Registry Notion DBs
- **Decided:** archived (moved out of Sales Hub via MCP). Routines never reference these. Liam right-clicks → Delete in Notion UI for permadelete.
- **Why:** Activity Log dedupe (now Attio Notes title-prefix dedupe) replaces these.

## Operational rules

### Always commit to main, no session branches
- **Decided:** All work commits directly to `main`. No ephemeral session branches.
- **Why:** Cloud routines read `main` by default. Session branches cause stale-content confusion.
- **Operator action required:** delete leftover `claude/<adjective>-<id>` remote branches via GitHub UI when they appear.

### Cloud routines fresh-clone each run
- **Decided:** Each routine run does fresh `git clone`, work, `git add memory/`, `git commit`, `git push origin main`.
- **Why:** No long-lived state on cloud side. Repo is the operational scratchpad.

### Coworker drops from 3x/day to 2x/day (Pro tier budget)
- **Decided:** `coworker` runs 2x weekdays (8am + 4pm), not 3x. Schedule total: 2 coworker + 1 daily-review = 3 routine runs/weekday + 1 self-audit/Sunday.
- **Why:** Pro plan caps at 5 routine runs/day. Original schedule (3 + 1 = 4 weekday) left only 1 ad-hoc slot, which is too tight for testing + manual triggers. 2 + 1 = 3 leaves 2 ad-hoc slots.
- **Trade-off:** worst-case latency from inbound (transcript / email) to first-pass draft is ~8 hours instead of ~4. Daily-review still prunes drafts the same evening, so total wait-and-prune cycle is unchanged.
- **Upgrade path:** Max plan (15 runs/day) if routine count grows or the latency becomes a problem.

### Activity Log is the dedupe ledger; no Skill State DB usage [SUPERSEDED]
- **Decided originally:** Routines query the Notion Activity Log (embedding source IDs in Summary/Notes) to dedupe.
- **Superseded by:** Attio Notes title-prefix dedupe (May 2026 Attio pivot). Same principle, different storage.
