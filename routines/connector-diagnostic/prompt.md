# desk-monkey-connector-diagnostic

Manual, read-only. Verify that the Cloud Routine runtime exposes the Attio capabilities `skills/attio-tooling.md` requires. Write nothing to Attio. Report results to `memory/runlog.md`.

## Step 0 — Preconditions

1. Read `CLAUDE.md`.
2. Read `skills/attio-tooling.md`.
3. Use Attio only. Do not call any other connector for writes.

## Step 1 — Stub runlog

Append to `memory/runlog.md`:
```
## <ISO> — desk-monkey-connector-diagnostic [IN_PROGRESS]
```

## Step 2 — Inspect available Attio tools

Look at the Attio connector tools actually exposed in this routine runtime. Do NOT assume any specific tool name. Map what you see to these capabilities:

- search records
- list records
- get records by ids
- create record
- upsert record
- update record
- list attribute definitions
- create note
- list tasks
- create task
- update task
- list lists
- list list attribute definitions
- list records in list
- add record to list

For each capability: yes/no based on actual presence.

## Step 3 — Read-only Deal schema probe

Using the list-attribute-definitions capability for object `deals` (if available):

1. List Deal object attribute definitions.
2. Capture: required fields, all attribute slugs, and the allowed select options for Stage, Forecast Category, Service Line, Source, Deal Type, Buyer Behavior Stage, Champion Verdict.
3. Compare against the Selling With required attribute set in `CLAUDE.md`. Note any missing custom attributes — Liam needs to add them in the Attio UI.

If list-attribute-definitions is unavailable, skip this step and log `BLOCKED_TOOL_GAP: Attio list-attribute-definitions unavailable in this routine`.

## Step 4 — Read-only writes-availability probe

Do NOT actually call create-record, upsert-record, update-record, create-note, create-task, update-task, or add-record-to-list. Just confirm the tool is exposed and callable in principle.

If `create-record` for object `deals` appears unavailable, output:
```
BLOCKED_TOOL_GAP: Attio create-record unavailable for deals in Claude Routine runtime
```

Repeat for each capability that is missing relative to the runtime preflight list in `skills/attio-tooling.md`.

## Step 5 — Append capability summary to runlog

Append the compact tool-contract line:
```
TOOL_CONTRACT Attio: search=<yes/no> list=<yes/no> create=<yes/no> upsert=<yes/no> update=<yes/no> attrs=<yes/no> notes=<yes/no> tasks=<yes/no> lists=<yes/no>
```

Then append a short report:
```
### Deal schema present in workspace
- required Selling With attributes present: <count>/<expected count>
- missing Selling With attributes: [<list or "none">]
- Stage options seen: [<list>]
- Forecast Category options seen: [<list>]
```

## Step 6 — Final runlog + commit

Replace `[IN_PROGRESS]` with:

```
## <ISO> — desk-monkey-connector-diagnostic
**Expected:** read-only inspection of Attio tool surface in Cloud Routine runtime
**Actual:**
- Capabilities available: <count>/<expected>
- Missing capabilities: [<list or "none">]
- Deal schema check: <ok | gaps>
- BLOCKED_TOOL_GAP entries: <count>
**Status:** ✅ Healthy | ⚠️ Drift (missing optional capabilities) | 🔴 Failed (missing required capabilities)
**Operator action:** if status is 🔴, fix the Attio connector setup (re-add the hosted MCP at https://mcp.attio.com/mcp, re-OAuth, confirm the routine includes the Attio connector). If Deal custom attributes are missing, add them in the Attio UI per CLAUDE.md.
```

Commit: `git add memory/runlog.md && git commit -m "connector-diagnostic <ISO>" && git push origin main`.

## Hard rules

- READ-ONLY on Attio. No create / upsert / update / delete on any record, note, task, or list entry.
- No Gmail draft, no Calendar write, no Notion write.
- Do not invent tool names. Use only what the runtime actually exposes.
- ALWAYS write the runlog summary before exit, even on partial failure.
- Browser automation is forbidden.
