# self-audit

Cloud routine. Runs weekly (suggested Sundays 7pm). Reads the last 200 lines of `memory/runlog.md`, identifies recurring drift patterns and errors, sweeps Attio for stale Deals, writes findings to `memory/audit.md`. Read-only on the rest of the system.

## Trigger
- **Schedule:** set in the Anthropic routine UI (suggested `0 19 * * 0`)
- **Connectors needed:** Attio (stale-deal sweep + critical-finding Tasks)
- **Routine prompt:** `Read routines/self-audit/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## What it does
- Scans the last 200 lines of runlog (bounded — does not read the whole file)
- Groups by routine: counts ✅ Healthy / ⚠️ Drift / 🔴 Failed runs, dedupes errors and drift patterns
- Identifies recurring issues: same drift across 3+ runs, same error across 2+ runs, stuck IN_PROGRESS reports
- Sweeps Attio Deals for Last Touch >14d (active Stages only) and creates `[DEFCON 3] [Follow-up] [Liam] stale Deal: <name>` Tasks
- Appends findings to `memory/audit.md` with a `Week ending <date>` header
- For CRITICAL bugs (data loss, silent send failures, duplicate Notes, Tasks not landing): also creates a `[DEFCON 1] [Internal] [Liam]` Attio Task

## Hard rules
- READ-ONLY on prompts and skill files. Propose patches only. Liam applies.
- WRITE only to `memory/audit.md`, `memory/runlog.md`, Attio Tasks (for stale-deal + critical findings).
- Never re-audit a week already in audit.md. Check headers first.
