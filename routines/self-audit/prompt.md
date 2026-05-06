# desk-monkey-self-audit

Weekly Sundays 7pm. Read last 200 lines of runlog. Find drift patterns. Sweep stale Deals. Write findings to `memory/audit.md`.

## Step 0 — Idempotency + stub runlog

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

Query Deals (`collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4`) where Last Touch is more than 14 days ago AND Stage NOT IN [Closed Won, Closed Lost, Nurture, On Hold, Unresponsive].

For each stale Deal:
- Check DEFCON Tasks for an existing Open task with Task=`stale Deal: <name>` matching this Deal. Skip if present.
- Otherwise create a DEFCON Task: Task=`stale Deal: <name> — last touch <date>`, Owner=Liam, DEFCON=3, Category=Follow-up, Source=Internal, Deal relation set, Notes=`14+ day silence; recent Activity Log entries: <count>; current Stage: <stage>`.

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

If you see CRITICAL bugs (data loss, sends firing without intent, view-filter blindness, duplicate Activity Log rows, DEFCON Tasks not landing, runlog corrupted):
- Create a DEFCON Task: Task=`Self-audit critical finding: <one-line>`, Owner=Liam, DEFCON=1, Category=Internal, Source=Internal, Notes=link to relevant runlog entries.

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
- WRITE only to `memory/audit.md`, `memory/runlog.md`, DEFCON Tasks (for stale-deal + critical findings).
- Never re-audit a week already covered. The audit.md header check is the only guard needed.
- Never read more than the last 200 lines of runlog.
