# Skill: attio-tooling

Canonical Attio tool contract. Loaded by every routine that touches Attio. Read this before any Attio read or write.

## Purpose

Use Attio as the canonical CRM without assuming exact MCP function names. At runtime, use the actual Attio connector tools exposed to the routine. The repo describes desired capabilities; the runtime decides which connector tool fulfills them.

## Hard rules

- Never call or write instructions that depend on fake implementation names like `mcp__Attio__create_record`.
- Never assume `assert_record` exists. Prefer Attio hosted MCP semantics: `upsert-record` when available, otherwise `search-records` + `create-record` / `update-record`.
- Never write Deal fields before calling `list-attribute-definitions` for object `deals`.
- Never write list-entry fields before calling `list-list-attribute-definitions` for the target list.
- Never create duplicate People, Companies, Deals, Notes, or Tasks. Search first.
- If a required write tool is unavailable, do not invent a workaround. Log `BLOCKED_TOOL_GAP`.
- Browser automation is forbidden for Attio writes unless Liam explicitly approves it in a later prompt.
- Zapier MCP may be used only as a fallback for Attio if direct Attio hosted MCP is unavailable or missing one required write action.

## Verified Attio hosted MCP capability surface

Source: Attio hosted MCP at `https://mcp.attio.com/mcp` (OAuth). At time of writing, this MCP documents:

- `search-records`
- `list-records`
- `get-records-by-ids`
- `create-record`
- `upsert-record`
- `update-record`
- `list-attribute-definitions`
- `list-lists`
- `list-list-attribute-definitions`
- `list-records-in-list`
- `add-record-to-list`
- `create-note`
- `list-tasks`
- `create-task`
- `update-task`

Routines must verify these are present at runtime before relying on them. The list above is documentation, not a runtime guarantee.

## Runtime preflight

At the start of every routine that needs Attio:

1. Inspect the available Attio connector tools.
2. Confirm whether these capabilities exist:
   - search records
   - list records
   - create record
   - upsert record
   - update record
   - list attribute definitions
   - create note
   - list tasks
   - create task
   - update task
   - list lists
   - add record to list
3. Append a compact tool-contract line to `memory/runlog.md`:
   ```
   TOOL_CONTRACT Attio: search=<yes/no> list=<yes/no> create=<yes/no> upsert=<yes/no> update=<yes/no> attrs=<yes/no> notes=<yes/no> tasks=<yes/no> lists=<yes/no>
   ```
4. If a required capability is missing for the routine's task, append:
   ```
   BLOCKED_TOOL_GAP: Attio <capability> unavailable in this routine
   ```
   Then skip only the blocked write section. Continue safe reads and summaries.

## Deal creation protocol

Only create a Deal when there is strong buying signal:

- real sales meeting
- pricing/proposal request
- POC discussion
- migration/implementation discussion
- renewal pain
- explicit next step with a customer
- Liam manually asked for a Deal

If confidence is below 0.80, do not create a Deal. Create a Liam-owned Attio Task instead:

```
[DEFCON 3] [Pipeline Build] [Liam] Review whether to create Deal for <Company>
```

Steps to create a Deal:

1. Search for existing Company by domain/name (search records / list records capability).
2. Upsert Company if missing and enough data exists (upsert record capability if available; otherwise search → create / update).
3. Search for existing Person by email.
4. Upsert Person if missing and enough data exists.
5. Search existing Deals for same company/person/topic and active stage.
6. If a likely active Deal exists, update that Deal instead of creating a duplicate.
7. Call `list-attribute-definitions` for object `deals`. Capture the actual attribute slugs and the allowed select options for Stage, Forecast Category, Service Line, Source, Deal Type, Buyer Behavior Stage, Champion Verdict.
8. Build a Deal attribute map using only actual fields and allowed select options. If a value from the source has no matching option, omit that field and log `FIELD_OPTION_GAP`.
9. Create the Deal with `create-record` for object `deals`, or use `upsert-record` if there is a reliable unique matching attribute.
10. Attach a Note explaining why the Deal was created (create-note capability).
11. Create a Liam-owned follow-up Task (create-task capability).
12. If a relevant sales pipeline list exists and the tool is available, call `list-list-attribute-definitions` for the target list, then `add-record-to-list`.
13. Append the result to `memory/runlog.md`.

## Deal update protocol

- Use the update-record capability, not any fake `update_record_attributes` tool name.
- Never overwrite a populated field with blank data.
- Never auto-advance Stage. If a stage gate is met, create a Task for Liam instead.
- Always append to Deal Evolution. Newest item goes at top. Never erase history.
- Use hard evidence only.

## People + Companies write protocol

- People match key: email address. Search first; if missing, create or upsert.
- Companies match key: domain first, name as fallback. Search first; if missing, create or upsert.
- For both: never overwrite a populated attribute with blank data. Only fill blanks.
- Custom select fields require `list-attribute-definitions` first to confirm allowed options.

## Notes + Tasks write protocol

- Notes: dedupe by title-prefix source ID before creating. Use the create-note capability.
- Tasks: dedupe by linked Deal + title prefix (Category + Owner + action keywords) before creating. Use the create-task capability for new ones, update-task for existing.
- Counterpart-owned tasks: assignees field empty, Owner name in title prefix, verification path in content.

## Pipeline list membership

Adding a record to a sales pipeline list is a separate action from creating the Deal record itself. Sequence:

1. `list-lists` to find the target list.
2. `list-list-attribute-definitions` to discover required entry attributes.
3. `add-record-to-list` with the Deal record ID and any required entry attributes.

If list-lists or add-record-to-list is unavailable in the runtime, log `BLOCKED_TOOL_GAP: Attio list membership unavailable` and continue without the list assignment. The Deal record still exists.

## Zapier fallback

If direct Attio hosted MCP lacks a needed action, use Zapier MCP only for:

- Create or Update Record
- Create Record
- Update Record
- Create Note
- Create Task
- Create or Update List Entry by Parent Record
- Create List Entry

Do not make Zapier the primary Attio path. Direct Attio hosted MCP first; Zapier only if a specific capability is missing.

## When everything is blocked

If the routine cannot complete its CRM work because required Attio write tools are unavailable:

1. Log `BLOCKED_TOOL_GAP: Attio <capability> unavailable in this routine` for each missing capability.
2. Continue safe reads (search-records, list-records, list-tasks, list-attribute-definitions) and summaries.
3. Skip only the blocked write section. Do not abort the whole routine.
4. Surface the gap clearly in the final runlog report so Liam can fix the connector setup.
