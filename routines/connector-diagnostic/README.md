# connector-diagnostic

Manual routine. Read-only. Confirms a Cloud Routine sees the same Attio tools that a normal Claude chat sees.

Must run before `contact-migration` or any production CRM writes. Cloud Routines have a different connector surface than chat — capabilities present in chat are not guaranteed in a routine. This routine writes nothing to Attio. It only inspects, lists, and reports.

## Trigger
- **Schedule:** none. Manual one-shot from the Anthropic routine UI.
- **Connectors needed:** Attio (direct hosted MCP at `https://mcp.attio.com/mcp`).
- **Routine prompt:** `Read routines/connector-diagnostic/prompt.md and follow it exactly. Read CLAUDE.md and skills/* first for context.`

## What it does

- Reads `CLAUDE.md` and `skills/attio-tooling.md`.
- Inspects available Attio connector tools at runtime.
- Reports whether the capabilities listed in `skills/attio-tooling.md` runtime preflight are available.
- Lists Deal object attribute definitions (required fields, allowed Stage / status options).
- Appends a `TOOL_CONTRACT` line and a capability summary to `memory/runlog.md`.
- If `create-record` for object `deals` is unavailable, logs `BLOCKED_TOOL_GAP: Attio create-record unavailable for deals in Claude Routine runtime`.

## What it does NOT do

- No create / update / delete on any Attio object.
- No Gmail draft, no Calendar write, no Notion write.
- No commits other than the runlog summary.

## When to run

- Before the very first `contact-migration` run.
- After any Attio connector reauthentication.
- After Anthropic announces connector-surface changes.
- Any time `coworker` or `daily-review` logs `BLOCKED_TOOL_GAP` for Attio.

## See also
- `prompt.md` — exact steps
- `../../skills/attio-tooling.md` — the runtime tool contract this routine validates
- `../../CLAUDE.md` — working memory
