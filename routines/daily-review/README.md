# daily-review

Once-daily evening rollup. Three jobs:

1. **Counterpart commitment verification.** Sweep DEFCON Tasks past their Due with Owner=blank. Search Gmail/Calendar for proof of completion. If found, mark Done. If not, create a Liam-owned nudge Task and mark the original Blocked.
2. **Deal property updates rolled up from today.** Query Activity Log for today's logged activity per Deal. Apply Selling With deal property updates (Champion Tests, Buyer Behavior Stage, Next Action, Identified Pain, etc.) on hard evidence. Append a one-line Deal Evolution entry per deal.
3. **Left-on-read prospecting.** Active early-stage Deals with no reply >7d → create a soft-nudge DEFCON Task for Liam.

## Trigger
- **Schedule:** set in the Anthropic routine UI (suggested `30 18 * * 1-5` — 6:30pm weekdays after the 6pm coworker run)
- **Connectors:** Notion, Gmail, Google Calendar, Zapier (for any per-message label work)
- **Routine prompt:** `Read routines/daily-review/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## Why separate from coworker

coworker runs 3x daily and is tactical (log + draft + DEFCON Tasks for the past 24h). daily-review is the strategic rollup: takes the day's accumulated Activity Log entries, decides what changed at the Deal level, and runs the longer sweeps (counterpart verification + prospecting) that don't need to repeat 3x/day.

## See also
- `prompt.md`
- `../coworker/` — the tactical loop this rollups
- `../../CLAUDE.md`
