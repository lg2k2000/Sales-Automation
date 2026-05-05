# self-audit

Cloud routine. Runs weekly (suggested Sundays 7pm). Reads the last 200 lines of `memory/runlog.md`, identifies recurring drift patterns and errors, writes findings to `memory/audit.md`. Read-only on the rest of the system.

## Trigger
- **Schedule:** set in the Anthropic routine UI (suggested `0 19 * * 0`)
- **Connectors needed:** Notion (only for critical-finding escalation)
- **Routine prompt:** `Read routines/self-audit/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## What it does
- Scans the last 200 lines of runlog (bounded — does not read the whole file)
- Groups by skill: counts ✅ Healthy / ⚠️ Drift / 🔴 Failed runs, dedupes errors and drift patterns
- Identifies recurring issues: same drift across 3+ runs, same error across 2+ runs, stuck IN_PROGRESS reports
- Appends findings to `memory/audit.md` with a `Week ending <date>` header
- For CRITICAL bugs (data loss, silent send failures, view-filter blindness): also writes a Notion Activity Log row (Channel=Note, Direction=Internal, Status=Open Action) so it surfaces in Liam's 🔨 To-Dos view

## Hard rules
- READ-ONLY on everything except `memory/audit.md` and `memory/runlog.md` (and the critical-finding Notion row)
- Never modifies prompt files. Proposes patches only. Liam applies them.
- Never re-audits a week already in audit.md. Checks headers first.
