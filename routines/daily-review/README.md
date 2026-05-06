# daily-review

Once-daily evening rollup. Four jobs:

1. **Prune today's first-pass meeting drafts** (anti-AI-pacing). Find Attio Notes from today with `[first-pass draft pending prune]` markers. Rewrite the linked Gmail draft to 30-50% length. Replace the draft. Update the Note marker to `[draft pruned <ISO>]`.
2. **Counterpart commitment verification.** Sweep Attio Tasks past their deadline_at with empty assignees. Search Gmail/Calendar for proof of completion. If found, mark Task complete. If not, create a Liam-owned nudge Task and mark the original [BLOCKED].
3. **Deal property updates rolled up from today.** Query Attio Notes from today linked to a Deal. Apply Selling With Deal Record updates (Champion Tests, Buyer Behavior Stage, Next Action, Identified Pain, etc.) on hard evidence. Append a one-line Deal Evolution entry per deal.
4. **Left-on-read prospecting.** Active early-stage Deals with no reply >7d → create a soft-nudge Attio Task for Liam.

## Trigger
- **Schedule:** set in the Anthropic routine UI (suggested `30 18 * * 1-5` — 6:30pm weekdays after the 4pm coworker run)
- **Connectors:** Attio, Notion (Projects only), Gmail, Google Calendar, Zapier (for any per-message label work)
- **Routine prompt:** `Read routines/daily-review/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## Why separate from coworker

coworker runs 2x daily and is tactical (log + draft + Attio Tasks for the past 24h). daily-review is the strategic rollup: takes the day's accumulated Attio Notes, decides what changed at the Deal level, and runs the longer sweeps (counterpart verification + prospecting) that don't need to repeat 2x/day.

## See also
- `prompt.md`
- `../coworker/` — the tactical loop this rollups
- `../../CLAUDE.md`
