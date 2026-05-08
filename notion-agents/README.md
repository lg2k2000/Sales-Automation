# Notion Agents build pack

Six files, each focused on one concern. Load whichever you need; each is independently usable.

| File | What's in it | When to load |
|---|---|---|
| `voice-rules.md` | Style guide: how to write so Liam doesn't catch an AI tell. Compressed humanizer rules, signature spec, scheduling-as-offer, read-aloud test. | Any time an agent (or you) drafts content for Liam or his counterparts. |
| `banned-list.md` | Hard no-fly list: banned vocab, banned phrases, banned punctuation, banned patterns. | Same — pair with voice-rules. |
| `reference-ids.md` | Every Notion UUID, page ID, draft ID, Attio ID, calendar event ID, Slack ID. Single source of truth. | Whenever you need to navigate to or write to a specific resource. |
| `phased-plan.md` | Six-phase build sequence with "go" criteria between each phase. | Before kicking off the build. Reference at every phase boundary. |
| `agent-specs.md` | Six AI Agent specs: name, job, inputs, outputs, trigger, scope, tools, instructions, test. | Phase 3 (deploy agents). |
| `operating-constraints.md` | Hard NEVERs, destructive-action gates, idempotency rules, rate limits, OAuth scopes, allowlists, stage lifecycles, run log spec. | Read once before starting; reference whenever an agent or tool is about to do something risky. |

## How to use

**If you're a browser-control agent (Claude Computer Use / Claude Chrome / Comet) building this:** load all six in order. The `phased-plan.md` is your roadmap. Confirm Phase 0 with Liam before crossing any boundary.

**If you're a researcher comparing tools:** the `phased-plan.md` and `agent-specs.md` together describe the job. Use them to evaluate which automation tool can actually do this in 2026.

**If you're Liam:** open the Daily Brief once it exists. Talk to it. The agents handle the rest.

## Plan-of-record

- Attio retired 2026-05-07. Notion canonical for Deals, Contacts, Activity Log, Projects.
- Six Notion 3.0 AI Agents replace the prior Claude Code `assistant` cron routine.
- Voice rules + banned list apply to every external draft.
- Hard NEVERs at top of `operating-constraints.md` are non-negotiable.

## Open decision points (default value in parens; ask Liam to confirm or override)

1. **Email + Calendar feed cadence** (hourly scheduled sweep) — alternative: Zapier/Make webhook for real-time
2. **Daily Brief shape** (single overwriting page) — alternative: Briefs DB with one row per brief for history
3. **Repo fate** (demote to docs) — alternative: kill repo, or keep it and route runlog there too
4. **Slack sidecar** (off — Notion comments on Daily Brief is the talkback surface) — alternative: keep Slack `#all-desk-monkey` as a parallel surface

## Latest sweep findings (2026-05-07, 14:00 MT)

- 4 CRM databases live in Notion with full Selling With + MEDPICC schema (Deals, Contacts, Activity Log, Projects). Currently flagged "legacy"; Phase 1 reactivates them.
- 17 skill templates already exist on the 🚧 Master Notion Agent page (Skill: Deal Update, Skill: Follow-up Email, Skill: Pipeline Audit, etc.). Reference these from agent prompts; don't rewrite.
- Notion native meeting transcription is active. ~40 transcripts in the past month. Drop Fireflies entirely.
- ~10 records in Attio need migration (HFP USA Deal, Andrew Brink, Anthony Orlovsky, three Liam tasks, two BRIEF Notes; Craig Walicek already exists in Notion Contacts).
- 5 Gmail drafts sitting in Liam's Gmail Drafts from today's session — kept; not deleted.
- 6th draft (Friday May 15 dialer windows to Miles) obsolete after Liam's May 14 invite — Liam to delete manually.
