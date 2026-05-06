# Design Decisions Log

Captures architectural decisions made during the build of Desk Monkey. Each entry: what got decided, why, what got rejected.

## Brain layer

### Notion is canonical, not Postgres
- **Decided:** Notion holds all business state (Activity Log, DEFCON Tasks, Deals, Contacts, Projects). No separate database.
- **Why:** Notion is already Liam's dashboard. Two databases means sync hell. Data volume doesn't justify Postgres.
- **Rejected:** Hosted Postgres. Local SQLite. Skill State Notion DB as a separate state mechanism.

### Activity Log doubles as dedupe ledger
- **Decided:** Before processing a transcript or email thread, query Activity Log for an existing row referencing the source ID. If present, skip.
- **Why:** Single source of truth. No `state.json` to maintain. No cursors to lose.
- **Rejected:** `memory/state.json` with `last_processed_id`. Notion Skill State DB key-value table.

### Two cadences, four routines
- **Decided:** `coworker` (3x weekdays, tactical), `daily-review` (1x evening, rollup), `self-audit` (weekly), `contact-migration` (manual one-shot).
- **Why:** Tactical vs strategic split keeps the fast loop fast. Counterpart verification + Deal property rollups need a calmer pass; tactical sweep can't wait on those.
- **Rejected:** Single mega-routine. Real-time event-driven (would require webhooks).

### Polling, not webhooks
- **Decided:** Routines pull from Fireflies / Gmail / Calendar on a schedule.
- **Why:** Webhooks add infra complexity for marginal latency improvement. Hours of latency is fine for follow-up work; minutes don't matter.

## Tool layer

### Direct MCPs primary, Zapier fills gaps
- **Decided:** Use direct MCPs for read AND write whenever they exist. Zapier only for actions direct MCPs don't expose.
- **Why:** Direct MCPs are faster, free per-call, more structured. Zapier costs Zap tasks per call and has less control over response shape.
- **Currently filling gaps via Zapier:** Gmail Send / Archive / Trash / Mark Read/Unread / Add Label / Remove Label, Google Contacts CRUD.

### Gmail filter creation is not exposed; coworker does it
- **Decided:** `coworker` Step 2a sweeps inbox every 4 hours and labels + archives system noise (newsletters, receipts, notifications) via Zapier `Add Label` and `Archive Email`.
- **Why:** Neither direct Gmail MCP nor Zapier expose `users.settings.filters.create`. Routine-based approximation is the next-best option.
- **Trade-off:** Worst-case 4-hour delay before noise gets out of inbox. Acceptable.

### Fireflies for transcripts
- **Decided:** Fireflies Pro plan ($18/mo). Bot joins meetings via calendar invite, transcribes regardless of platform.
- **Why:** Works as a guest on others' meetings (Google Meet recording requires host). Has direct MCP. Returns structured summaries + action items.
- **Rejected:** Granola (needs your Mac in the meeting), Otter / Read.ai (no MCP).

## Risk controls

### Drafts only, never autonomous sends
- **Decided:** Routines never call Zapier `Send Email`. Email drafts go to Gmail Drafts via direct `create_draft`. Liam clicks Send.
- **Why:** Single point of human review. Embarrassing-send risk concentrated in one click that Liam controls.
- **Calendar invites with attendees:** routines never auto-create. They surface a DEFCON Task; Liam creates the event manually.

### Wait-and-prune for meeting follow-ups
- **Decided:** `coworker` writes a first-pass draft within hours of meeting end. `daily-review` (evening) prunes it to 30-50% length and replaces the draft. Liam sees the pruned version next morning.
- **Why:** A draft sent 30 seconds after a meeting reads as AI. A tightened draft the next morning reads as a person.
- **Marker:** `[first-pass draft pending prune]` in Activity Log Action Items, replaced with `[draft pruned <ISO>]` after prune pass.

### Voice: polite AND concise, not either/or
- **Decided:** Hunter S. Thompson honesty + Hemingway brevity, BUT polite. No commanding language at the recipient. No "Pick one. Cut Y. Pull Z." even if short.
- **Why:** Brevity without politeness reads as bossy. Liam's actual voice is direct + polite, not direct + aggressive.
- **Reframings:** "Could you...", "Would you mind...", "Happy to talk through it."

### Email signature mandatory
- **Decided:** Every email draft ends with the full Liam Glennie / phone / website signature block. First-name-only sign-offs not acceptable.
- **Why:** Looks professional, not like an AI bot impersonating Liam.

## Inbox strategy

### Role-based labels, not deal-based
- **Decided:** `Customer`, `Prospect`, `Partner`, `Vendor`, `Personal` plus state labels `_Action`, `_Waiting`, `_Reference` plus `System/Receipts`, `System/Newsletters`, `System/Notifications`. 11 labels total, all timeless.
- **Why:** Roles persist for years; deals come and go. Use Gmail search for company-specific finds.
- **Rejected:** One label per active Deal (`Houston-Foam`, `Anthropic-Identity`, etc.) — accumulates cruft.

### Coworker auto-labels by relationship type
- **Decided:** When `coworker` processes a Gmail thread tied to a Notion Contact, it applies the relationship label based on Deal Stage (Closed Won → Customer; active stages → Prospect; Referrer → Partner) plus a state label (`_Action` or `_Waiting`).
- **Why:** Removes manual labeling from Liam's daily routine.

## What got rejected wholesale

### iMessage triage / send-worker (deferred, not in current build)
- **Decided not to wire up:** Mac stdio MCP for iMessage (mac_messages_mcp). Local cron-based triage and send-worker.
- **Why:** Liam decided not to set up the local Mac infrastructure for now. iMessage triage can return later if priorities change. Activity Log Channel=Text and Status=Send Now remain in the Notion schema but no current routine uses them.

### Local Claude Code MCP config (`.mcp.json`)
- **Decided not to keep:** Liam isn't running Claude Code locally on this repo. The file added cognitive overhead with no benefit.

### Slack as inbound channel
- **Status:** Discussed, not implemented. Slack MCP via Zapier is doable (~30 min setup) but defer until Notion-Activity-Log-as-inbox proves insufficient.

## Operational rules

### Always commit to main, no session branches
- **Decided:** All work commits directly to `main`. No ephemeral session branches.
- **Why:** Cloud routines read `main` by default. Session branches cause stale-content confusion.
- **Operator action required:** delete leftover `claude/<adjective>-<id>` remote branches via GitHub UI when they appear.

### Cloud routines fresh-clone each run
- **Decided:** Each routine run does fresh `git clone`, work, `git add memory/`, `git commit`, `git push origin main`.
- **Why:** No long-lived state on cloud side. Repo is the operational scratchpad.
