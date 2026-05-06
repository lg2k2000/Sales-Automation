# desk-monkey-self-audit

Weekly Sundays 7pm. Read last 200 lines of runlog. Find drift patterns. Sweep stale Deals. Write findings to `memory/audit.md`.

## Step 0a — Read the Attio tool contract

Read `skills/attio-tooling.md`. Run the Attio runtime preflight before any CRM action. Append the `TOOL_CONTRACT` line to `memory/runlog.md`. If the Attio create-task or list-records capability is unavailable, log `BLOCKED_TOOL_GAP` and route stale-deal findings to `memory/audit.md` and `memory/runlog.md` instead of failing or attempting a write that won't succeed.

Do NOT use browser automation for Attio. Do NOT hardcode MCP function names — use the connector tools the runtime actually exposes.

## Step 0b — Idempotency + stub runlog

Check `memory/audit.md` for an existing `## Week ending <YYYY-MM-DD>` header for the current ISO week. If present, exit immediately — already audited.

Append to runlog:
```
## <ISO> — desk-monkey-self-audit [IN_PROGRESS]
```

## Step 1 — Read runlog tail (BOUNDED)

Read only the last 200 lines of `memory/runlog.md`. Do NOT read the whole file.

## Step 2 — Group + count by routine

For each routine (coworker / daily-review / contact-migration):
- Total runs in the window
- Counts of ✅ Healthy / ⚠️ Drift / 🔴 Failed
- Distinct error types (deduped by message)
- Distinct drift patterns (deduped by phrasing)
- Stuck `[IN_PROGRESS]` entries (routine died mid-run)

## Step 3 — Identify recurring patterns

Flag for prompt patch:
- Same drift across 3+ runs
- Same error across 2+ runs
- Stuck `[IN_PROGRESS]` (any count)
- Patterns Liam called out in chat that the routine kept regressing on (humanizer voice slip, missing Notion context, dropped action items, duplicated rows)

## Step 4 — Stale Deal sweep

Query Attio Deals via the Attio list-records / search-records capability where Last Touch is more than 14 days ago AND Stage NOT IN [Closed Won, Closed Lost, Nurture, On Hold, Unresponsive].

For each stale Deal:
- Check Attio Tasks via the list-tasks capability for an existing Open task with title `[DEFCON 3] [Follow-up] [Liam] stale Deal: <name>` matching this Deal. Skip if present.
- Otherwise create an Attio Task via the create-task capability: content=`[DEFCON 3] [Follow-up] [Liam] stale Deal: <name> — last touch <date>`, deadline_at=today+2, assignees=[liam@deskmonkeyai.com], linked_records=[Deal], description=`14+ day silence; recent Notes count: <N>; current Stage: <stage>`.

If create-task is unavailable in the runtime, do NOT fail the routine. Append the stale-deal list to the audit.md "Stale Deals surfaced this week" section and to the runlog. Liam triages from there manually.

## Step 5 — Write findings to audit.md

Append a new section. The existing-week-header check in Step 0 prevents duplicates.

```
## Week ending <YYYY-MM-DD>

### Recurring drift (worth a prompt patch)
1. [pattern] — observed in <N> runs across <routine> — proposed patch: <one-line>

### One-off errors (don't patch)
- ...

### Healthy patterns to preserve
- ...

### Stale Deals surfaced this week
- <Deal name> (last touch <date>, stage <stage>) — DEFCON Task created
- ...
```

## Step 6 — Critical-bug escalation

If you see CRITICAL bugs (data loss, sends firing without intent, view-filter blindness, duplicate Attio Notes, Tasks not landing, runlog corrupted):
- Create an Attio Task via the create-task capability: content=`[DEFCON 1] [Internal] [Liam] Self-audit critical finding: <one-line>`, deadline_at=today, assignees=[liam@deskmonkeyai.com], description=link to relevant runlog entries.
- If create-task is unavailable, surface the finding inline in `memory/audit.md` under a `### CRITICAL findings (no Attio Task — runtime gap)` section and in the runlog with a `BLOCKED_TOOL_GAP` note.

## Step 7 — Final runlog + commit

Replace `[IN_PROGRESS]` with:

```
## <ISO> — desk-monkey-self-audit
**Expected:** scan last 200 lines runlog, identify drift, stale-deal sweep, audit.md update
**Actual:**
- Lines scanned: <N>
- Recurring drift patterns: <N>
- One-off errors: <N>
- Stuck IN_PROGRESS: <N>
- Stale Deals surfaced: <N>
- Critical findings escalated: <N>
**Status:** ✅ Healthy
```

Commit: `git add memory/runlog.md memory/audit.md && git commit -m "self-audit week <ISO-week>" && git push origin main`.

## Hard rules

- READ-ONLY on all prompts and skill files. Propose patches in `audit.md`. Liam applies them.
- WRITE only to `memory/audit.md`, `memory/runlog.md`, Attio Tasks (for stale-deal + critical findings).
- Never re-audit a week already covered. The audit.md header check is the only guard needed.
- Never read more than the last 200 lines of runlog.
- Never hardcode Attio MCP function names. Use capability names from `skills/attio-tooling.md`.
- Never use browser automation.
- If the Attio task-create capability is missing, route findings into `memory/audit.md` instead of failing.
