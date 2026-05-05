# desk-monkey-self-audit

You run weekly Sundays 7pm. Read last 200 lines of runlog. Identify drift. Sweep stale Deals. Write findings to `memory/audit.md`. Read CLAUDE.md before starting.

## Step 0 — Idempotency + stub runlog

Query Skill State DB for Skill=self-audit, Key=`last_audit_week`. If Value matches the current week (ISO week number), exit immediately — don't re-audit a week already covered.

Append to runlog:
```
## <ISO> — desk-monkey-self-audit [IN_PROGRESS]
```

## Step 1 — Read runlog tail (BOUNDED)

Read only the last 200 lines of `memory/runlog.md`. Never read the whole file.

## Step 2 — Group + count by skill

For each skill (coworker / contact-migration / triage / send-worker):
- Total runs
- Counts of ✅ Healthy / ⚠️ Drift / 🔴 Failed
- Distinct error types (deduped by message)
- Distinct drift patterns (deduped by phrasing)
- Stuck `[IN_PROGRESS]` entries (skill died mid-run)

## Step 3 — Identify recurring patterns

Flag for prompt patch:
- Same drift across 3+ runs
- Same error across 2+ runs
- Stuck `[IN_PROGRESS]` (any count)
- Patterns Liam called out in chat that the skill kept regressing on (humanizer voice slip, missing Notion context, redundant questions, dropped Action Items)

## Step 4 — Stale Deal sweep

Query Deals (`collection://5f892d2b-0d1d-40e2-a6d0-0cd3be6a83c4`) where Last Touch is more than 14 days ago AND Stage NOT IN [Closed Won, Closed Lost, Nurture, On Hold, Unresponsive].

For each stale Deal:
- Create a DEFCON Task (`collection://e16abae8-b4c2-4cb1-a41c-0b53a583a44e`): Task=`stale Deal: <name> — last touch <date>`, Owner=Liam, DEFCON=3, Category=Follow-up, Source=Internal, Deal relation set, Notes=`14+ day silence; last activity in <Activity Log link>`.
- Skip if a Task with this exact title already exists Open.

## Step 5 — Write findings to audit.md

Append a section:

```
## Week ending <YYYY-MM-DD>

### Recurring drift (worth a prompt patch)
1. [pattern] — observed in <N> runs across <skill> — proposed patch: <one-line>

### One-off errors (don't patch)
- ...

### Healthy patterns to preserve
- ...

### Stale Deals surfaced this week
- <Deal name> (last touch <date>) — DEFCON Task created
- ...
```

## Step 6 — Critical-bug escalation

If you see CRITICAL bugs (data loss, sends failing silently, view-filter blindness, duplicate Activity Log rows, DEFCON Tasks not landing), also create a DEFCON Task:
- Task=`Self-audit critical finding: <one-line>`
- Owner=Liam, DEFCON=1, Category=Internal, Source=Internal, Notes=link to relevant runlog entries

## Step 7 — Mark week audited + final runlog

Upsert Skill State: Skill=self-audit, Key=`last_audit_week`, Value=current ISO week (e.g. `2026-W18`).

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
- READ-ONLY on prompts and SKILL.md files. Propose patches in audit.md. Liam applies.
- WRITE-ONLY on `memory/audit.md`, `memory/runlog.md`, DEFCON Tasks (for stale-deal + critical findings), Skill State (for `last_audit_week`).
- Never re-audit a week already in audit.md or covered by `last_audit_week`.
- Never read the full runlog. 200 lines max.
