# deal-updater

Cloud Claude Code routine. Runs daily at 7am weekdays. Reads Notion Activity Log for the past 24 hours, updates Deal page properties only when conversational evidence is high-confidence.

## Trigger

- **Schedule:** `0 7 * * 1-5`
- **Repo:** desk-monkey-routines
- **Connectors:** notion
- **Routine prompt:** `Read routines/deal-updater/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## What it does

Walks new Activity Log entries (Calls, Meetings, or substantive Notes) that have a Deal relation and haven't been processed yet. For each entry, decides whether the conversation provides hard evidence to update fields on the linked Deal: Stage, Next Action, Identified Pain, Champion Verdict, Compelling Event, Cost of Doing Nothing, Decision Process, Open Questions, Future State.

Always appends a one-line Deal Evolution entry. Marks the Activity Log entry processed by appending `[Deal updated YYYY-MM-DD]` to Action Items. Writes a final report to `memory/runlog.md`.

## Hard NEVERs

- Never advances Stage without explicit verbal commitment
- Never invents facts not in the entry
- Never overwrites Deal Evolution (always append)
- Never touches Forecast Category — that stays Liam's call

## See also

- `prompt.md` — exact instructions executed by the routine
- `../../CLAUDE.md` — working memory loaded into every session
- `../../memory/runlog.md` — run history
