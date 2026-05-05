# desk-monkey-self-audit

You run Sundays at 7pm. Read past week of runlog entries. Identify drift patterns. Write findings + prompt-patch proposals to memory/audit.md. Read-only on the system, write-only on audit.md.

## Step 0 — Stub the runlog
Append to memory/runlog.md:
```
## <ISO> — desk-monkey-self-audit [IN_PROGRESS]
```

## Step 1 — Read runlog (BOUNDED)
Read only the last 200 lines of memory/runlog.md. Do NOT read the whole file.

## Step 2 — Group + count by skill
For each skill (triage, send-worker, deal-updater):
- Total runs
- Runs marked ✅ Healthy / ⚠️ Drift / 🔴 Failed
- Distinct error types (deduped by message)
- Distinct drift patterns (deduped by phrasing)

## Step 3 — Identify recurring patterns
Look for:
- Same drift entry across 3+ runs
- Same error across 2+ runs
- Skills that wrote no final report (stuck IN_PROGRESS)
- Patterns Liam called out in chat that the skill kept regressing on (humanizer voice, missing Notion context, redundant questions)

## Step 4 — Write findings to audit.md
Append a section to memory/audit.md:
```
## Week ending <YYYY-MM-DD>

### Recurring drift (worth a prompt patch)
1. [pattern] — observed in <N> runs across <skill> — proposed patch: <one-line>
2. ...

### One-off errors (don't patch)
- ...

### Healthy patterns to preserve
- ...
```

## Step 5 — Critical-bug escalation
If you see a CRITICAL bug (data loss, sends failing silently, view-filter blindness), also create a Notion Activity Log row:
- Channel=Note, Direction=Internal, Status=Open Action
- Activity="Self-audit critical finding: <one-line>"
- Action Items=link to the relevant runlog entries

This surfaces it in Liam's 🔨 To-Dos view immediately. Don't wait for him to read audit.md.

## Step 6 — Final runlog report
Replace [IN_PROGRESS] with:
```
## <ISO> — desk-monkey-self-audit
**Expected:** scan last 200 lines runlog, identify drift, write findings
**Actual:**
- Lines scanned: <N>
- Recurring drift patterns: <N>
- One-off errors: <N>
- Critical findings escalated to Notion: <N>
**Status:** ✅ Healthy
```

## Hard rules
- READ-ONLY on everything except memory/audit.md and memory/runlog.md (and the critical-finding Notion row)
- Never modify SKILL.md or prompt files yourself — propose patches, Liam applies
- Never re-audit a week already audited (check audit.md headers first)
